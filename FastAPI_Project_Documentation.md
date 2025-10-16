# 📘 FastAPI Project Documentation

## 📁 Project Structure
```
/srv/fastapi
├─ app/
│  ├─ main.py
│  ├─ api/
│  │  ├─ deps.py
│  │  ├─ routers/
│  │  │  ├─ health.py
│  │  │  └─ v1/
│  │  │     └─ items.py
│  ├─ core/
│  │  ├─ config.py
│  │  ├─ logging.py
│  │  └─ security.py
│  ├─ db/
│  │  ├─ base.py
│  │  ├─ session.py
│  │  └─ migrations/
│  ├─ models/
│  │  └─ item.py
│  ├─ schemas/
│  │  └─ item.py
│  ├─ services/
│  │  └─ items.py
│  └─ middleware/
│     └─ request_id.py
├─ .env
├─ pyproject.toml / requirements.txt
└─ gunicorn_conf.py
```

---

## 🧠 1. Overview
This project is a **modular FastAPI backend** designed for scalability and maintainability.  
It follows **Clean Architecture** principles with separate layers for:

- **Models** — SQLAlchemy ORM models  
- **Schemas** — Pydantic models for request/response validation  
- **Services** — Business logic and CRUD operations  
- **Routers** — API endpoints  
- **Core** — Application configuration, logging, and security  
- **Database** — Connection handling and migrations  
- **Middleware** — Custom request/response handlers  

---

## ⚙️ 2. Setup Instructions

### Create virtual environment
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate   # Windows
```

### Install dependencies
```bash
pip install -r requirements.txt
```

### Create `.env` file
```
DATABASE_URL=sqlite:///./test.db
PROJECT_NAME=FastAPI Demo
```

### Initialize Alembic (optional)
```bash
cd app/db
alembic init migrations
```

---

## 📁 3. File-by-File Purpose

| File | Purpose |
|------|----------|
| **main.py** | Entry point for FastAPI. Includes routers. |
| **api/routers/** | API routes grouped by version/module. |
| **core/config.py** | App configuration and environment. |
| **db/session.py** | Database connection and session setup. |
| **models/item.py** | SQLAlchemy model for `Item`. |
| **schemas/item.py** | Pydantic schemas for validation. |
| **services/items.py** | CRUD logic for items. |
| **middleware/request_id.py** | Custom middleware example. |
| **gunicorn_conf.py** | Gunicorn config for production. |

---

## 🔁 4. Development Flow

| Step | File | Purpose |
|------|------|----------|
| 1️⃣ | `models/` | Define DB model |
| 2️⃣ | `schemas/` | Define Pydantic schema |
| 3️⃣ | `services/` | Add CRUD logic |
| 4️⃣ | `api/routers/` | Create endpoints |
| 5️⃣ | `main.py` | Include router |
| 6️⃣ | `db/base.py` | Register model |
| 7️⃣ | Alembic | Run migration |

### Flow Summary:
```
1️⃣ Model → 2️⃣ Schema → 3️⃣ Service → 4️⃣ Router → 5️⃣ main.py → 6️⃣ Migrate
```

---

## 🚀 5. Run and Test

### Start server
```bash
uvicorn app.main:app --reload
```

### Access docs
- Swagger: http://127.0.0.1:8000/docs  
- ReDoc: http://127.0.0.1:8000/redoc

### Test endpoints
**GET** `/health/` → `{ "status": "ok" }`  
**POST** `/api/v1/items/` → create item  
**GET** `/api/v1/items/` → list items

---

## 🧰 6. Gunicorn Setup
```python
# gunicorn_conf.py
import multiprocessing
workers = multiprocessing.cpu_count() * 2 + 1
bind = "0.0.0.0:8000"
worker_class = "uvicorn.workers.UvicornWorker"
timeout = 120
```

Run:
```bash
gunicorn app.main:app -c gunicorn_conf.py
```

---

## 🧾 7. Folder Responsibilities

| Folder | Responsibility |
|--------|----------------|
| `api/` | Defines all routes |
| `core/` | Config, logging, security |
| `db/` | DB session and migrations |
| `models/` | SQLAlchemy tables |
| `schemas/` | Pydantic models |
| `services/` | Business logic |
| `middleware/` | Request/response intercepts |

---

## 🧩 8. Optional Enhancements
- ✅ JWT authentication
- ✅ Role-based access
- ✅ Logging middleware
- ✅ Celery background tasks
- ✅ Monitoring (Sentry/Prometheus)
