# Golf App (Scaffold)

Monorepo with:
- **backend/**: FastAPI + SQLAlchemy + Alembic
- **frontend/**: Vite + React + TypeScript
- **spec/**: AI spec bundle location (contracts, rules, tests)

## Prereqs
- Docker + Docker Compose
- Python 3.11+ (recommended)
- Node.js 18+ (recommended)

## Quick start (local)

### 1) Start Postgres
```bash
cp .env.example .env
docker compose up -d
```

### 2) Backend setup
```bash
cd backend
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt -r requirements-dev.txt

# Create the initial migration (first time only)
alembic revision --autogenerate -m "init"
alembic upgrade head

# Run API
uvicorn app.main:app --reload
```

API will be at: http://localhost:8000  
Docs: http://localhost:8000/docs

### 3) Frontend setup
```bash
cd frontend
npm install
npm run dev
```

Frontend will be at: http://localhost:5173

### 4) Tests
```bash
cd backend
pytest
```

## Notes
- Configure environment variables via `.env` (see `.env.example`).
- The backend uses `DATABASE_URL` from the environment. In Docker Compose, it points at the `db` service.
- Put your AI spec bundle files under `spec/` (or replace the placeholders).

## Suggested next steps
1. Define SQLAlchemy models in `backend/app/db/models/`
2. Generate migrations with Alembic
3. Implement domain logic in `backend/app/services/`
4. Add API routes in `backend/app/api/routers/`
5. Add frontend routes/pages and connect to backend API