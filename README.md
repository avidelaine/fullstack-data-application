# Collaborative calendar — FastAPI · PostgreSQL · Streamlit

A full-stack calendar application built end-to-end: a typed REST API with JWT authentication, a relational schema with seed data, a web front end, and a three-level test suite — all wired together with Docker Compose.

Final project of the *Full-stack data application* course at ESIEE Paris (5th year, Nov–Dec 2025), built with Antoine Ritz.

![Architecture](img/Architecture.png)

## Stack

| Layer | Folder | Tech |
|---|---|---|
| API | `01-api/` | FastAPI, SQLAlchemy, Pydantic |
| Database | `02-database/` | PostgreSQL 16, schema + seed SQL applied at first start |
| Front end | `03-webapp/` | Streamlit |
| Auth | `04-authentication/` | PBKDF2 password hashing (stdlib `hashlib`), JWT (HS256) access tokens |
| Tests | `05-tests/` | pytest — unit, integration, end-to-end |
| Seed | `06-seed/` | Random users/events filler (Compose profile `tools`) |

## Run it

```bash
git clone https://github.com/avidelaine/fullstack-data-application.git
cd fullstack-data-application
cp .env.example .env          # then set POSTGRES_PASSWORD, JWT_SECRET_KEY, COOKIES_PASSWORD
docker compose up --build
```

| Service | URL |
|---|---|
| Web app | http://localhost:3000 |
| API docs (Swagger) | http://localhost:8000/docs |

Optional: `docker compose --profile tools run filler` to generate extra users and events.

## API

| Method | Route | Auth | Purpose |
|---|---|---|---|
| POST | `/auth/register` | – | Create a user |
| POST | `/auth/login` | – | Get a JWT |
| GET | `/events` | JWT | List the current user's events |
| POST | `/events` | JWT | Create an event |
| GET | `/events/{id}` | JWT | Read one event |
| PUT | `/events/{id}` | JWT | Update an event |
| DELETE | `/events/{id}` | JWT | Delete an event |
| GET | `/health`, `/health/db` | – | Liveness; database connectivity |

Every `/events` route is scoped to the authenticated user: you cannot read or modify someone else's events, and the integration tests check it.

## Tests

```bash
pip install -r 05-tests/requirements.txt
pytest -m unit                 # no database needed
pytest -m integration          # needs the db container
pytest -m e2e                  # needs the full stack up
```

35 tests: unit (hashing, token creation/validation, repository functions), integration (auth routes, database round-trips, route protection), end-to-end (API reachable from outside the Compose network).

## Data model

![Classes](img/Classes.png)

## Notes

Secrets are read from `.env`, which is git-ignored; `.env.example` lists the variables. The Streamlit front end talks to the API over the Compose network (`WEBAPP_INTERNAL_API_URL`), not through the browser.
