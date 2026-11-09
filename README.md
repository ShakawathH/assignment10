# Assignment 10 – Secure User Model, Pydantic Schemas, Tests & CI/CD

A FastAPI application with a calculator API and a secure SQLAlchemy `User` model
(bcrypt-hashed passwords, unique `username` / `email`), validated with Pydantic
schemas, tested with pytest + Playwright, and deployed through GitHub Actions to Docker Hub.

**Docker Hub repository:** https://hub.docker.com/r/shakawathh/assignment10

## Project structure

| Path | Purpose |
| --- | --- |
| `app/models/user.py` | SQLAlchemy `User` model (hashed passwords, unique username/email, JWT helpers) |
| `app/schemas/` | Pydantic schemas: `UserCreate`, `UserRead` (alias of `UserResponse`), `UserLogin`, `Token` |
| `app/auth/dependencies.py` | Current-user dependencies |
| `app/operations/` | Calculator operations |
| `main.py` | FastAPI app (`/`, `/add`, `/subtract`, `/multiply`, `/divide`, `/health`) |
| `tests/unit`, `tests/integration`, `tests/e2e` | pytest suites |
| `.github/workflows/test.yml` | CI/CD: test -> Trivy scan -> Docker Hub deploy |

## Run the tests locally

1. Create a virtual environment and install dependencies:

   ```bash
   python3 -m venv venv
   source venv/bin/activate        # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   playwright install chromium
   ```

2. Start PostgreSQL (the tests need it, default URL `postgresql://postgres:postgres@localhost:5432/fastapi_db`):

   ```bash
   docker compose up -d db
   ```

   If the container exits with a data-directory error, reset the old volume:
   `docker compose down -v && docker compose up -d db`.

3. Run everything:

   ```bash
   pytest
   ```

   Or by suite: `pytest tests/unit/`, `pytest tests/integration/`, `pytest tests/e2e/`.
   Extra options: `--run-slow` (include slow tests), `--preserve-db` (keep data after the run).

## Run the application

```bash
python main.py            # http://127.0.0.1:8000
# or with Docker Compose
docker compose up --build
```

## Run the published Docker image

```bash
docker pull shakawathh/assignment10:latest
docker run -p 8000:8000 shakawathh/assignment10:latest
```

Then open http://localhost:8000 (health check: http://localhost:8000/health).

## CI/CD pipeline

On every push / pull request to `main`, GitHub Actions:

1. **test** – starts a PostgreSQL 16 service, installs dependencies, runs unit, integration and e2e tests.
2. **security** – builds the Docker image and scans it with Trivy (fails on fixable HIGH/CRITICAL issues).
3. **deploy** – on `main` only, builds and pushes the image to Docker Hub.

Required repository secrets: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`.
