# CBSbest-app - AI PROJECT STACK & CONVENTIONS

This project uses a fully local, no-subscription development stack.
When generating code, setup steps, or architecture, follow these standards unless explicitly told otherwise.

⸻

CORE STACK

Backend
	•	Python 3.11 or newer
	•	FastAPI
	•	SQLAlchemy (2.0 style ORM)
	•	Psycopg (psycopg3 driver)
	•	Alembic for database migrations
	•	Uvicorn as the local development server

Database
	•	PostgreSQL 16 running locally in Docker
	•	Managed with docker compose
	•	No hosted or cloud database services

Environment
	•	macOS development
	•	Docker runtime via Colima
	•	Python virtual environments using venv

⸻

ARCHITECTURE RULES
	•	Backend-first API architecture
	•	Standard flow:
FastAPI → Service Layer → SQLAlchemy ORM → PostgreSQL
	•	Prefer SQLAlchemy ORM over raw SQL
	•	All schema changes must go through Alembic migrations
	•	Configuration must come from environment variables

⸻

PROJECT STRUCTURE

Use this layout:

project-root/
docker-compose.yml
.env
backend/
app/
main.py
db.py
models/
schemas/
routes/
services/
alembic/
alembic.ini
pyproject.toml
requirements.txt (optional)

⸻

DATABASE CONFIGURATION

Assume Postgres runs with:
Host: localhost
Port: 5432
Database: appdb
User: app
Password: app

SQLAlchemy connection string:
postgresql+psycopg://app:app@localhost:5432/appdb

Do not suggest cloud database providers.

⸻

DOCKER EXPECTATIONS

docker-compose.yml should include:
	•	postgres:16 container
	•	Named volume for persistent storage
	•	Healthcheck using pg_isready
	•	Optional pgadmin container is acceptable

Do not introduce additional services unless requested.

⸻

BACKEND STANDARDS

FastAPI
	•	Use APIRouter per feature or module
	•	Use Pydantic models for request and response schemas
	•	Use dependency injection for database sessions

Database Session Pattern

engine = create_engine(DATABASE_URL, pool_pre_ping=True)
SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False)

Provide a request-scoped session dependency.

⸻

MIGRATIONS (REQUIRED)

All schema changes must use Alembic.

Standard workflow:
alembic revision –autogenerate -m “description”
alembic upgrade head

Never instruct manual schema edits.

⸻

RUNNING THE APP (DEVELOPMENT)

Start the backend with:
uvicorn app.main:app –reload –port 8000

API documentation is available at:
http://localhost:8000/docs

⸻

DEPENDENCY MANAGEMENT

Preferred setup:

python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt

Or install via pyproject.toml:
pip install .

Do not default to Poetry, Pipenv, or Conda.

⸻

ENVIRONMENT VARIABLES

Use a .env file containing:
DATABASE_URL=postgresql+psycopg://app:app@localhost:5432/appdb

Use python-dotenv only if environment loading is needed.

⸻

FRONTEND (ONLY IF REQUESTED)

If a frontend is required:
	•	Prefer server-rendered templates
OR
	•	Minimal Vite + React setup

Do not introduce Next.js, Firebase, or SSR frameworks unless asked.

⸻

TESTING (WHEN NEEDED)
	•	Use pytest
	•	Prefer transactional test patterns or a dedicated test database
	•	Avoid heavy containerized test setups unless explicitly requested

⸻

WHAT TO AVOID

Do not:
	•	Use hosted database platforms
	•	Require SaaS accounts
	•	Add Kubernetes or cloud infrastructure
	•	Over-engineer with microservices

Keep the stack local, simple, and developer-run.

⸻

GUIDING PRINCIPLE

This project is for learning and rapid local development.
Favor clarity, minimal dependencies, and reproducible setup over production-scale complexity.