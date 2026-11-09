+++
title = "How I Hacked FastAPI BackgroundTasks for Real-Time Progress Updates (Without Celery)"
date = 2025-10-01
description = "Building a FastAPI endpoint that runs work in the background and reports real-time progress, without Celery."
draft = false

[extra]
toc = true
og_image = "cover.jpg"
+++

I Built a FastAPI Endpoint That Runs in the Background and Talks Back

![Image was generated with ChatGPT](cover.jpg)

*Image was generated with ChatGPT*

## I Built a FastAPI Endpoint That Runs in the Background and Talks Back

## Intro

Recently, at work, I had to build a set of REST API endpoints to trigger an AI model process that could take several minutes to complete. The usual options weren’t ideal:

- **Long-polling** from the frontend until the result is ready (wasteful).
- **Blocking** the request until the model finishes (terrible UX).
- **Fire-and-forget background job** with an email notification at the end (better, but not interactive).

Clearly, running the process in the background is efficient since it frees up the application to handle other requests. But here’s the problem: I wanted to *peek into the current state of the background process* and report progress back to the client—without committing to a heavyweight background manager like Celery.

## Problem

You kick off an AI model through a FastAPI endpoint. It takes minutes. Your frontend can either keep hammering with polls, freeze the request until it’s done, or dump the job into a background worker and email the result later.

None of those felt right for me. I wanted **real-time progress updates**—without bringing in Celery, Redis, or Kubernetes-sized machinery. So I hacked FastAPI’s `BackgroundTasks` into whispering back status updates while it ran. Here’s how.

FastAPI supports background tasks through the `BackgroundTasks` dependency. But it’s fire-and-forget. Once a task starts, you get no insight into its progress or result in real-time. Need to peek behind the curtain? You're out of luck—unless you add some extra plumbing.

## Solution

FastApi comes with a BackgroundTasks for managing Background tasks in a route, but it posses no native way of interacting with those background task once they have been initiated.

below is a simple example of a route that initiates a background task that prints the numbers to a million

```python
# main.py
import time
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def count_to(n: int, job_id: str, update_progress: Callable[..., Awaitable[None]]) -> int:
    """
    Iterates from 1 up to n and returns the values of n
    This is meant to be used to represent a slow running function.
    """
    result = 1
    for i in range(1, n):
        update_progress(job_id, "processing", round((i / n) * 100, 2))
        print(f"{i}")
        result = i

    update_progress(job_id, "completed", 100, result, ended_at=time.time())
    return result

@app.post("/tasks")
async def initiate_task(background_task: BackgroundTasks):
    background_task.add_task(count_to)
    return {"detail": "Task Started Successfully"}
```

You can run the FastAPI application with the followings commands

```bash
fastapi dev main.py
```

if you visit the endpoint you would immediately get a response that says

```bash
curl localhost:8000/tasks -X POST

# {"detail":"Task Started Successfully"}%  
```

And if you look at your application logs, you would see that numbers were printed from 1 through 999999

This shows that the route started the count_to function in the background and immediately returned a message to the client

Now, There is no direct way to obtain information about the running background task and report to the client (this can be setup with Redis and other Background process tools but for a simple case like us).

I achieved this by making a global data structure to keep track of all the background task so that at any point after the background task has been started I can refer to this data structure to get update about the current state of the task and possibly it result

```python
import uuid 
import time
from typing import Callable, Awaitable

from fastapi import FastAPI, BackgroundTasks, HTTPException


app = FastAPI()

task_jobs = {}


def count_to(n: int, job_id: str, update_progress: Callable[..., int]) -> int:
    """
    Iterates from 1 up to n and returns the values of n
    This is meant to be used to represent a slow running function.
    """
    result = 1
    for i in range(1, n):
        update_progress(job_id, "processing", round((i / n) * 100, 2))
        print(f"{i}")
        result = i

    update_progress(job_id, "completed", 100, result, ended_at=time.time())
    return result


@app.post("/tasks")
async def initiate_task(background_task: BackgroundTasks):
    job_id = str(uuid.uuid4())

    # register the task to the task db
    task_jobs[job_id] = {
        "status": "initiated",
        "progress": 0.0,
        "result": None,
        "errors": None,
    }

    def update_task(job_id: str, status: str, progress: float | int, result = None, errors = None):
        if not job_id in task_jobs:
            return
        
        task_jobs.update({
          "status": status,
          "progress": progress,
        })

        if result:
            task_jobs[job_id]["result"] = result
        if errors:
            task_jobs[job_id]["errors"] = errors

    background_task.add_task(count_to, 1_000_000, job_id, update_task)
    return {"message": "task started", "job_id": job_id}


@app.get("/tasks/status")
async def get_task_status(job_id: str):
    if not job_id in task_jobs:
        raise HTTPException(404, detail="Job ID not found")
    
    return task_jobs[job_id]
```

## Demo: Checking Task Status

Trigger a job:

```bash
curl -X POST localhost:8000/tasks
# {"message":"task started", "job_id":"c32cfe25-7b9c-4e87-83ec-80e7d6dcbf9e"}
```

Now check its progress:

```bash
curl "localhost:8000/tasks/status?job_id=c32cfe25-7b9c-4e87-83ec-80e7d6dcbf9e"
```

Sample response:

```json
{
  "status": "processing",
  "progress": 42.7,
  "result": null,
  "errors": null,
  "ended_at": null
}
```

When it finishes:

```json
{
  "status": "completed",
  "progress": 100,
  "result": 999999,
  "errors": null,
  "ended_at": 1727783087.12345
}
```

Now your background task is no longer mute—it talks back through the global dictionary.

The route called initiate_task is a post route, that generate a unique id on each call to be able to track each task individually, the default task structure is registered in the task database (dict in our case), an inner function is defined within the route, this is important because this function would be passed into the background task as an argument to the function (task) we are tying to process in the background, now it is up to you to call the update_progress functions at different points in your background task function to update the progress of the function execution, This way, the background functions talks back through the global data structure (dict)on it current state.

> “That’s it. Your FastAPI now moonlights as a background worker. Tell your boss you just saved $10K in server costs—or buy yourself snacks. You earned it.”

## Potential Issues

> ⚠️ *Heads up:* `BackgroundTasks.add_task` expects a **sync function**. If you pass it an `async def`, it won’t be awaited properly, which means FastAPI will just launch it and hope for the best. If you need `async`, consider using a task queue like Celery, RQ, or a custom loop via `asyncio.create_task` (outside of `BackgroundTasks`).

And just like that, your humble FastAPI app can now juggle tasks in the background *and* whisper back progress updates like a responsible magician. You didn’t need Celery. You didn’t need Redis. You just needed a dict, a UUID, and a stubborn refusal to give up.

> 💡 *Pro tip: Don’t forget to eventually replace that dict with Redis or some other reliable data store, or something else persistent, if you go to production. RAM amnesia is real.*
