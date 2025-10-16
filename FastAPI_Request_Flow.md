# ⚙️ FastAPI Request Flow — Step-by-Step

Let's assume a client sends:
```
GET /api/v1/items/
```

---

## 🧭 1️⃣ Entry Point — `app/main.py`

### What happens
- Server starts using `uvicorn app.main:app --reload`
- Initializes the FastAPI app
- Imports and registers routers
- Loads DB setup and middleware

**File:** `app/main.py`

**Role:** Central hub that binds everything together.

---

## 🌐 2️⃣ Router Registration — `app/api/routers/v1/items.py`

When a request comes to `/api/v1/items/`, FastAPI matches the route and sends it here.

**File:** `app/api/routers/v1/items.py`

**Role:** Defines endpoints using `@router.get`, `@router.post`, etc.

---

## 🧩 3️⃣ Dependency Injection — `get_db()`

Before executing endpoint logic, FastAPI runs dependencies.

**File:** `app/api/routers/v1/items.py`

**Role:** Creates and yields a SQLAlchemy DB session.

---

## 🧠 4️⃣ Business Logic — `app/services/items.py`

Router calls the corresponding service function (e.g., `get_items(db)`).

**File:** `app/services/items.py`

**Role:** Handles CRUD and database operations.

---

## 🧱 5️⃣ Database ORM — `app/models/item.py`

Service layer interacts with SQLAlchemy model to fetch or modify data.

**File:** `app/models/item.py`

**Role:** Defines SQLAlchemy ORM mapping for tables.

---

## 📄 6️⃣ Schema Validation — `app/schemas/item.py`

Response data passes through a Pydantic schema before sending to client.

**File:** `app/schemas/item.py`

**Role:** Validates and serializes the response.

---

## 🧰 7️⃣ Middleware (Optional) — `app/middleware/request_id.py`

Middleware executes before and after every request.

**File:** `app/middleware/request_id.py`

**Role:** Logging, adding request IDs, timing, etc.

---

## 🧾 8️⃣ Response — Back to Client

Response is converted to JSON and sent back to the client.

---

# 🔁 Complete Request Flow Summary

| Step | File | Purpose |
|------|------|----------|
| 1️⃣ | **main.py** | Initializes app, loads routers/middleware |
| 2️⃣ | **api/routers/v1/items.py** | Endpoint definition |
| 3️⃣ | **api/routers/v1/items.py → get_db()** | Creates DB session |
| 4️⃣ | **services/items.py** | Business logic |
| 5️⃣ | **models/item.py** | ORM model |
| 6️⃣ | **schemas/item.py** | Schema validation |
| 7️⃣ | **middleware/request_id.py** | Middleware execution |
| 8️⃣ | **main.py** | Response handling |

---

# 📊 Visual Flow Diagram

```
Client Request (GET /api/v1/items/)
        │
        ▼
main.py ───► Registers Routers + Middleware
        │
        ▼
api/routers/v1/items.py ───► @router.get("/")
        │
        ▼
get_db() ───► Opens DB session
        │
        ▼
services/items.py ───► Executes get_items()
        │
        ▼
models/item.py ───► SQLAlchemy ORM query
        │
        ▼
schemas/item.py ───► Pydantic response validation
        │
        ▼
middleware/request_id.py ───► Adds metadata/logs (optional)
        │
        ▼
Response JSON sent back to client
```

---

# ✅ Summary

**Full Request Lifecycle:**
```
main.py → router → dependency → service → model → schema → middleware → response
```
