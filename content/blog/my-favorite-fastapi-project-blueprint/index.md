+++
title = "My FastAPI Project Blueprint"
date = 2025-10-14
description = "The FastAPI project structure I reach for on every new project — lessons learnt from working on large codebases."
draft = false

[extra]
toc = true
+++

> I've refined this structure in production environments serving over 500+ concurrent users; it scales effortlessly, stays sane to maintain, and keeps productivity at full throttle.

## TL;DR

If you just want to use the template FastAPI structure directly in your project, check out
[this GitHub repo](#) for the complete project structure.

## Introduction

I have worked with FastAPI almost daily in the last year. During that period, I developed a
standard project structure that works exceptionally well to keep me motivated to make changes
and add features easily, and it also scales incredibly.

In this post, I will break down every part of my FastAPI go-to architecture — the one I optimize
whenever I'm starting a new project or refactoring an old one. Whether you're trying to ship
ideas faster, organize your growing codebase, or harden your app's security, this structure will
help you build confidently and evolve your project like a pro.

## Project Structure

```
.
├── Makefile
├── logs/
│   └── app.log
└── app/
    ├── __init__.py
    ├── api_router.py
    ├── dependencies.py
    ├── logger.py
    ├── main.py
    ├── middlewares.py
    ├── settings.py
    ├── routers/
    │   ├── admin.py
    │   ├── auth.py
    │   └── products.py
    ├── schemas/
    │   ├── admin.py
    │   ├── auth.py
    │   └── products.py
    └── services/
        ├── admin.py
        ├── auth.py
        └── products.py
```

## The App Directory

The `app` directory is the engine and core of the application. It contains all the modules and
code needed to power the majority of use cases for a backend infrastructure. I'll explain each
directory and module in the app directory and justify their use, pointing out common issues and
highlighting steps taken to improve code quality and developer experience.

### `__init__.py`

It is standard practice to include an `__init__.py` file in every Python package; this helps to
declare a folder as a regular Python package.

### `api_router.py`

I use this as a central module for managing all my routers in my application. This enables me to
have features like version controlling with `/v1/users`, `/v2/users` while keeping the codebase
easy to follow and clean.

```python
# app/api_router.py

from fastapi import APIRouter

api_v1 = APIRouter(prefix="/v1")
api_v2 = APIRouter(prefix="/v2")

# include routes to a root route
# from app.routers import routers as user_routers
# api_v1.include_router(users_routers)
```

### `logger.py`

Logging is a crucial part of every software application. It gives visibility into the operation
of any running software application and can help guide resolution of issues and bugs when they
do happen — and they *will* happen.

Instead of getting overwhelmed with configurations, this template works for me in almost all my
projects. You can extend it by setting up an external handler to connect your log records to a
pretty dashboard or analytics tool. This particular configuration allows me to group logs by day,
achieved with the `TimedRotatingFileHandler`, making the process of debugging much easier.

```python
# app/logger.py

import logging
import json

from logging.handlers import TimedRotatingFileHandler

logger = logging.getLogger()


class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_record = {
            "timestamp": self.formatTime(record, self.datefmt),
            "level": record.levelname,
            "module": record.module,
            "funcName": record.funcName,
            "lineno": record.lineno,
            "message": record.getMessage(),
        }
        return json.dumps(log_record)


file_handler = TimedRotatingFileHandler(
    "logs/app.log", when="midnight", interval=1 / 86400, backupCount=7
)

file_handler.setFormatter(JsonFormatter())

logger.handlers = [file_handler]
logger.setLevel(logging.INFO)
```

### `main.py`

The main module is the entry point into the application. Most of the security policies and rules
for the application are set up here.

```python
# app/main.py

from contextlib import asynccontextmanager
from datetime import datetime, UTC
from fastapi import FastAPI

from starlette.middleware.trustedhost import TrustedHostMiddleware
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import RedirectResponse
from fastapi.responses import JSONResponse
from fastapi.requests import Request
from fastapi import HTTPException
from fastapi import FastAPI

from slowapi import Limiter
from slowapi.util import get_remote_address

from redis import asyncio as aioredis

from app.logger import logger
from app.api_router import api
from app.settings import Settings
from app.middlewares import log_request_middleware

settings = Settings()


@asynccontextmanager
async def lifespan(app: FastAPI):
    yield


def initiate_app():
    app = FastAPI(
        title="FastAPI Sample Project",
        summary="API for FastAPI Sample Project",
        lifespan=lifespan,
    )

    origins = [
        # Add allowed origins here
    ]

    app.add_middleware(
        CORSMiddleware,
        allow_origins=origins,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

    # tweak this to see the most efficient size
    app.add_middleware(GZipMiddleware, minimum_size=100)
    app.add_middleware(
        TrustedHostMiddleware,
        allowed_hosts=[
            # Add allowed hosts here, this controls host that the application can be hosted on
        ],
    )
    app.add_middleware(BaseHTTPMiddleware, dispatch=log_request_middleware)

    # this handles rate limiting protecting the API from abuse or Denial of Service Attacks
    limiter = Limiter(key_func=get_remote_address)
    app.state.limiter = limiter

    app.include_router(api)
    return app


app = initiate_app()


@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "detail": exc.detail,
            "path": request.url.path,
            "timestamp": datetime.now(UTC).isoformat(),
        },
    )


@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"Unexpected error: {str(exc)}")

    return JSONResponse(
        status_code=500,
        content={
            "detail": "An unexpected error occurred",
            "path": request.url.path,
        },
    )


@app.get("/", tags=["Root"])
async def root():
    # this redirects root requests to the docs page
    return RedirectResponse("/docs")
```

### `settings.py`

I have discovered that using Pydantic's `BaseSettings` model makes handling of environment
variables easy and clean. Types can be defined for the environment variables and they are
validated at app startup time, and default values can be provided for optional environment
variables.

```python
# app/settings.py

from pydantic_settings import BaseSettings
from pydantic import ConfigDict, AnyUrl


class Settings(BaseSettings):
    model_config = ConfigDict(env_file=".env")

    example_secret: str = "example secret value"

    JWT_SECRET: str  # required environment variable
    JWT_ALGORITHM: str = "HS256"  # optional environment variable with default value
```

### `dependencies.py`

I use this module to group my custom route dependencies. Common dependencies like `get_db`,
which is commonly used to get a session to interact with the database, can be declared here.
Using a dedicated module for dependencies makes code organization better and more modular, since
usually different routes depend on these dependencies. Testing is also easier since logic can be
tested in isolation.

### `middlewares.py`

I use this module to group all the custom middlewares I define for my application. The modular
benefits outlined for a dedicated dependencies module also apply here.

## Directories

### Routers

I find that creating a module for each logical grouping of routes makes development lovely and
easier — for example, an `auth.py` module for all routes relating to user authentication and
profile management, a `products.py` module for all routes relating to product management, and
`admin.py` for all routes related to the admin API.

I try to keep my route functions to at most 2 lines of code. The route function simply lays out
the route and defines the dependencies and query parameters needed, while the logic for each
route function is defined in a service function.

```python
# app/routers/auth.py

from typing import Annotated

from pydantic import EmailStr
from fastapi.routing import APIRouter
from sqlalchemy.ext.asyncio import AsyncSession
from fastapi import Depends, Body, BackgroundTasks

from app.dependencies import get_db
from app.schemas import auth as auth_schemas
from app.services import auth as auth_services

router = APIRouter(prefix="/auth", tags=["Authentication"])


EmailBody = Annotated[EmailStr, Body(embed=True)]
DBDep = Annotated[AsyncSession, Depends(get_db)]


@router.post("/signup", response_model=auth_schemas.UserModel)
async def signup(
    db: DBDep,
    bg_task: BackgroundTasks,
    request_data: auth_schemas.UserSignUpData,
):
    return await auth_services.signup_user(request_data, db, bg_task)
```

The route function is easy on the eyes: the business logic is handled in the service function,
while the route focuses on defining the dependencies and requirements in the form of a JSON body
or query parameters.

### Schemas

All my Pydantic models go into the schema module. Just like the routers directory, the schemas
directory houses all the schema modules. Usually I match each route module with a corresponding
schema module and a service module — this way each component of my application is grouped in a
very logical and predictable way. An example of a schema module is `auth.py`, shown below.

```python
# app/schemas/auth.py

from uuid import UUID
from datetime import datetime
from typing import Annotated

from pydantic import BaseModel, EmailStr, Field


class UserSignUpData(BaseModel):
    password: Annotated[str, Field(min_length=8)]
    email: Annotated[EmailStr, Field(max_length=254)]


class UserModel(BaseModel):
    id: UUID

    email: EmailStr
    date_created: datetime
    date_updated: datetime
```

### Services

The services directory houses modules with the actual business logic to process the backend
functionality. This can include API calls to third-party services, functions to make query calls
to the database, and so on. Separating them this way allows a major separation of concerns
between the route's API requirements and the actual implementation.

And just like I do for the routers and schemas directories, I match each module in those
directories with a corresponding module for the component covered in the services directory.
Working with the running example for the auth component:

```python
# app/services/auth.py

from datetime import timedelta

from fastapi import HTTPException
from passlib.context import CryptContext
from fastapi.security import OAuth2PasswordBearer

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.settings import Settings
from app.models import User as UserDB
from app.schemas import auth as auth_schema

settings = Settings()

JWT_SECRET = settings.JWT_SECRET
JWT_ALGORITHM = settings.JWT_ALGORITHM

ACCESS_TOKEN_LIFESPAN = timedelta(days=2)
REFRESH_TOKEN_LIFESPAN = timedelta(days=5)

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="v1/auth/token")


def verify_password(plain_password: str, hashed_password: str):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password: str):
    return pwd_context.hash(password)


async def get_user(email: str, session: AsyncSession) -> UserDB | None:
    stmt = select(UserDB).where(UserDB.email == email)
    result = await session.execute(stmt)
    return result.scalar_one_or_none()


async def create_user(
    user_data: auth_schema.UserSignUpData,
    session: AsyncSession,
):
    result = await session.execute(
        select(UserDB).where(UserDB.email == user_data.email)
    )
    if result.scalar_one_or_none():
        raise HTTPException(status_code=400, detail="Email already registered")

    result = await session.execute(
        select(UserDB).where(UserDB.email == user_data.email)
    )
    if result.scalar_one_or_none():
        raise HTTPException(status_code=400, detail="Username already used")

    hashed_password = get_password_hash(user_data.password)
    new_user = UserDB(
        email=user_data.email,
        password=hashed_password,
        username=user_data.email,
    )

    session.add(new_user)
    await session.commit()
    await session.refresh(new_user)
    return new_user


async def signup_user(
    data: auth_schema.UserSignUpData,
    session: AsyncSession,
):
    return await create_user(data, session)
```

## The Makefile

Outside the core backend Python concerns, I have come to appreciate making use of tools like
`make` to simplify invocation of common command line tools — like starting a FastAPI server,
starting a pytest session, and so on.

```makefile
# Makefile

run-local:
	fastapi dev app/main.py

test-local:
	pytest -s --cov

coverage-report:
	coverage report

coverage-html:
	coverage report && coverage html
```

In order to use the `make` tool you have to have `make` installed on your computer.

With this file in place at the root of my project, whenever I want to start my FastAPI server, I
can do so with the following command:

```sh
make run-local
```

Instead of typing out the full command. This contributes to increased productivity when running
certain command line tools over and over again.

## Conclusion

I ignored setup for database connections in this template.

Maintaining a modular project structure and codebase does more for you down the line than you
might think. A predictable and reasonable organization of functionality and tools gives beauty to
a codebase beyond the functionality itself.

Inasmuch as this pattern has improved my productivity in the past year, I actively seek to learn
and adopt better and better approaches to constructing and crafting my backend infrastructure.
