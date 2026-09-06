+++
title = "Projects"
description = "Things I've built, and things I've built in order to learn"
template = "projects.html"

[extra]
toc = true
github_user = "brianobot"

# ---- Featured work ------------------------------------------------------
[[extra.featured]]
name = "Pulse"
description = "A web service monitor — watches your endpoints and tells you the moment one stops answering."
url = "https://readpulse.app"
repo = ""

[[extra.featured]]
name = "FastAPI Project Structure"
description = "A production-ready FastAPI boilerplate enforcing clean architecture: structured layouts, JWT security, Pydantic v2 validation and scalable dependency injection."
repo = "brianobot/fastAPI_project_structure"

[[extra.featured]]
name = "Django Project Structure"
description = "The same idea for Django: clean architecture, JWT security, DRF, CORS headers, Spectacular and a custom user model, wired up and ready."
repo = "brianobot/django_project_structure"

[[extra.featured]]
name = "FastAPI Project Gen8"
description = "A lightweight CLI that scaffolds clean, production-ready FastAPI projects at warp speed."
repo = "brianobot/fastapi-project-gen8"

[[extra.featured]]
name = "HTTP Load Tester"
description = "A terminal UI for load-testing HTTP endpoints, written in Rust."
repo = "brianobot/http_load_tester"

[[extra.featured]]
name = "secwatch"
description = "A Rust command-line tool that scans REST APIs for common security vulnerabilities."
repo = "brianobot/secwatch"

# ---- Learning in public -------------------------------------------------
[[extra.learning]]
id = "python"
category = "Python"
blurb = "The language I reach for first, studied properly."
repos = [
  { name = "learning_asyncio_in_python", description = "Getting asyncio to make sense, concept by concept." },
  { name = "mastering-pydantic-for-python-developers", description = "Pydantic beyond the basics." },
  { name = "mastering_sqlalchemy", description = "SQLAlchemy studied against the official documentation." },
  { name = "mastering_django_models", description = "The Django model API, worked through from the docs." },
  { name = "mocking_in_tests_in_python", description = "Concepts learned while studying mocking in Python tests." },
  { name = "learning-hashmap-with-python", description = "Building a hashmap by hand to understand the real one." },
]

[[extra.learning]]
id = "rust"
category = "Rust"
blurb = "Where most of my low-level curiosity ends up."
repos = [
  { name = "learning_rust", description = "From hello world to fearless concurrency — daily progress, memory-safety breakthroughs and lessons learned." },
  { name = "rust_atomics_and_locks_low_level_currency_lessons", description = "Working through Rust Atomics and Locks, one concurrency primitive at a time." },
  { name = "learning-tokio-rs", description = "Documenting my way through the Tokio async runtime." },
  { name = "learning_axum-rs-", description = "Code, notes and small projects built while learning the Axum web framework." },
  { name = "learning_serde_rs", description = "Documenting how Serde actually does serialization in Rust." },
  { name = "comprehensive_rust_seminar", description = "Code and notes from Google's Comprehensive Rust seminar." },
  { name = "rust_cli_utilities", description = "Rebuilding common CLI utilities in Rust as a learning exercise." },
  { name = "rustlings-solved-", description = "My worked solutions to the Rustlings exercises." },
]

[[extra.learning]]
id = "systems"
category = "Systems & internals"
blurb = "Rebuilding things to find out how they actually work."
repos = [
  { name = "C_learning", description = "A comprehensive journey through C: memory management, pointers, data structures and low-level optimization." },
  { name = "py-lox", description = "The Lox programming language from Crafting Interpreters, implemented in Python." },
  { name = "redis-clone-python", description = "Redis, rebuilt from scratch in Python as a coding challenge." },
  { name = "codecrafters-redis-python", description = "The CodeCrafters 'build your own Redis' track." },
  { name = "learning-system-programming", description = "Notes and experiments in systems programming." },
  { name = "learning_data_structure_and_algorithm", description = "Data structures and algorithms, worked through from first principles." },
]
+++
