# InsightForge AI

InsightForge AI is a lightweight FastAPI application that acts as a base project for a knowledge intelligence platform. At the moment, it contains a minimal API scaffold with a health-check endpoint and a simple test suite. The application is intentionally easy to extend as more features, routes, and AI-powered services are added later.

## What this project does

The app starts by creating a FastAPI instance in [app/main.py](app/main.py). That instance is configured with:

- an app title: `InsightForge AI`
- a description: `A knowledge intelligence platform.`
- a version: `0.1.0`

The application exposes a single endpoint, `/health`, which returns a basic JSON response:

```json
{"status": "ok"}
```

This endpoint is useful for:

- confirming the app is running
- checking deployment health
- verifying startup in a container, VM, or local environment
- providing a simple baseline for future API services

## Setup requirements

Before running the project, make sure your local machine has the following:

- Python 3.12 or newer
- `pip` for installing Python dependencies
- a virtual environment tool such as `venv`
- access to the terminal or command prompt

## Project structure

```text
Insightforge-ai/
├── app/
│   ├── __init__.py
│   └── main.py
├── tests/
│   └── test_health.py
├── .gitignore
├── pyproject.toml
├── requirements.txt
├── README.md
├── .venv/   # local virtual environment (created by you)
└── .pytest_cache/  # created by tests when run locally
```

### Main files

- [app/main.py](app/main.py): defines the FastAPI application and API routes
- [tests/test_health.py](tests/test_health.py): validates the application health endpoint
- [requirements.txt](requirements.txt): lists the Python packages required by the project
- [pyproject.toml](pyproject.toml): project configuration for Python tooling and metadata

## How the project works internally

### 1. Application startup

When the app is started with `uvicorn`, Python loads the ASGI app from the module path `app.main:app`.

In [app/main.py](app/main.py), the code does the following:

```python
from fastapi import FastAPI

app = FastAPI(
    title="InsightForge AI",
    description="A knowledge intelligence platform.",
    version="0.1.0",
)
```

This creates the FastAPI application object that handles incoming web requests.

### 2. Route registration

A route is registered with the decorator:

```python
@app.get("/health")
def health_check() -> dict[str, str]:
    return {"status": "ok"}
```

This means:

- the HTTP method is `GET`
- the path is `/health`
- FastAPI will respond to that route automatically
- the returned dictionary is serialized as JSON

### 3. Request/response cycle

When a client sends a request to `/health`, the process is:

1. The ASGI server receives the HTTP request
2. FastAPI matches the request path to the registered route
3. The `health_check` function runs
4. The function returns `{"status": "ok"}`
5. FastAPI converts that Python dict into JSON
6. The response is sent back to the client with HTTP status `200 OK`

### 4. Testing flow

The test file [tests/test_health.py](tests/test_health.py) uses FastAPI's `TestClient` to simulate a browser or HTTP client without starting a server manually:

```python
client = TestClient(app)
response = client.get("/health")
```

The test then asserts:

- response status code is `200`
- response JSON matches `{"status": "ok"}`

This confirms the health endpoint is working correctly.

## How to run the project

### 1. Create a virtual environment

```bash
cd /workspaces/Insightforge-ai
python -m venv .venv
source .venv/bin/activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Start the web server

```bash
uvicorn app.main:app --reload
```

The app will start in development mode with auto-reload enabled.

## Local links and ports

By default, FastAPI/Uvicorn runs on port `8000`.

Use these URLs in your browser or API client:

- App root: http://127.0.0.1:8000
- Health endpoint: http://127.0.0.1:8000/health
- Swagger documentation: http://127.0.0.1:8000/docs
- ReDoc documentation: http://127.0.0.1:8000/redoc

### Example `curl` calls

```bash
curl http://127.0.0.1:8000/health
```

Expected output:

```json
{"status": "ok"}
```

## Running tests

To run the test suite:

```bash
pytest
```

This executes the checks in [tests/test_health.py](tests/test_health.py).

## API endpoints

### GET /health

Returns the current service health state.

Example response:

```json
{"status": "ok"}
```

HTTP status: `200 OK`

## Development notes

This project is intentionally minimal and serves as a foundation for future enhancements. As it grows, the same pattern can be extended with:

- additional route files
- database models and services
- authentication and authorization
- AI integrations and orchestration logic
- advanced analytics and reporting endpoints

## Summary

InsightForge AI is a small but solid FastAPI starter project. It provides a working base application, a health endpoint, and an automated test that verifies the service is responding correctly. It is designed to be easy to expand into a full knowledge intelligence platform.
