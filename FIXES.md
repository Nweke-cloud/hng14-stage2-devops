# Bug Fixes Documentation

## Fix 1
**File:** `api/.env`
**Line:** 1-2
**Problem:** Real credentials committed directly to the repository. The file contained `REDIS_PASSWORD=supersecretpassword123` in plain text in git history. Any public viewer could extract the password.
**Fix:** Removed file from git tracking with `git rm --cached api/.env`. Added `.env` to `.gitignore`. Created `.env.example` with placeholder values.

## Fix 2
**File:** `api/main.py`
**Line:** 8
**Problem:** Redis connection hardcoded to `host="localhost"`. Inside Docker, each container is its own network host. `localhost` inside the API container refers to the API container itself, not Redis. This causes a connection error on startup inside Docker.
**Fix:** Changed to read from environment variable: `host=os.getenv("REDIS_HOST", "redis")`

## Fix 3
**File:** `api/main.py`
**Line:** 8
**Problem:** Redis password defined in `.env` but never passed to the Redis client. Redis server configured with `--requirepass` would reject all connections without a password.
**Fix:** Added `password=os.getenv("REDIS_PASSWORD", None)` to Redis connection.

## Fix 4
**File:** `api/main.py`
**Line:** 8
**Problem:** Redis responses returned as bytes. Calling `.decode()` on every response is fragile and fails if `decode_responses` is not set.
**Fix:** Added `decode_responses=True` to Redis client. Removed manual `.decode()` calls.

## Fix 5
**File:** `worker/worker.py`
**Line:** 6
**Problem:** Redis connection hardcoded to `host="localhost"`. Same Docker networking issue as Fix 2. Worker cannot reach Redis inside a container network via localhost.
**Fix:** Changed to `host=os.getenv("REDIS_HOST", "redis")`

## Fix 6
**File:** `worker/worker.py`
**Line:** 6
**Problem:** Redis password not passed to client. Same issue as Fix 3.
**Fix:** Added `password=os.getenv("REDIS_PASSWORD", None)` to Redis connection.

## Fix 7
**File:** `worker/worker.py`
**Line:** 1-end
**Problem:** `signal` module imported but never used. The infinite `while True` loop has no graceful shutdown handler. When Docker sends SIGTERM to stop the container, the worker ignores it and gets killed forcefully mid-job, potentially leaving jobs in a broken state.
**Fix:** Added `handle_signal` function and registered handlers for SIGTERM and SIGINT. Replaced `while True` with `while running` so the loop exits cleanly on shutdown.

## Fix 8
**File:** `frontend/app.js`
**Line:** 6
**Problem:** API URL hardcoded to `http://localhost:8000`. Inside Docker, the frontend container cannot reach the API via localhost. Causes 500 errors on all job submission and status requests.
**Fix:** Changed to `process.env.API_URL || "http://api:8000"` to read from environment variable with correct Docker service name as default.

## Fix 9
**File:** `api/requirements.txt`, `worker/requirements.txt`
**Line:** all
**Problem:** No version pins on any dependency. Unpinned dependencies make builds non-reproducible — a build today may produce different behavior than a build next week if a package releases a breaking change.
**Fix:** Pinned all dependencies to specific versions: `fastapi==0.111.0`, `uvicorn==0.29.0`, `redis==5.0.4`.

## Fix 10
**File:** `api/` (missing), `worker/` (missing), `frontend/` (missing)
**Problem:** No `.dockerignore` files. Without them, `.env` files, `__pycache__`, and `node_modules` get copied into Docker images, leaking secrets and bloating image size.
**Fix:** Created `.dockerignore` for each service excluding `.env`, `__pycache__`, `*.pyc`, and `node_modules`.
