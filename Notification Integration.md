# Notification System Architecture (Redis + Scheduler + Worker)

This document describes the **asynchronous notification system architecture** implemented in the backend using:

* **FastAPI**
* **Redis Queue**
* **APScheduler**
* **Worker Process**
* **Firebase Cloud Messaging**

This design ensures **scalable, reliable, and non-blocking notification delivery**.

---

# 1. System Architecture Overview

The notification system follows a **producer-consumer architecture**.

```
FastAPI API
     │
     │ (create notification)
     ▼
APScheduler Job
     │
     │
     ▼
Notification Service
     │
     │ (push job)
     ▼
Redis Queue
     │
     │
     ▼
Notification Worker
     │
     │ (fetch tokens)
     ▼
Firebase Cloud Messaging
     │
     ▼
Mobile Devices
```

---

# 2. Components

## 2.1 FastAPI API Layer

Provides endpoints to manually send notifications.

### File

```
app/api/notification_router.py
```

### Code

```python
from fastapi import APIRouter
from app.services.firebase_service import send_push_notification, send_bulk_notification

router = APIRouter()

@router.post("/send-notification")
async def send_notification(token: str, title: str, body: str):

    result = await send_push_notification(
        token=token,
        title=title,
        body=body
    )

    return result


@router.post("/send-bulk-notification")
async def send_bulk_notification_api(tokens: list[str]):

    result = await send_bulk_notification(
        tokens=tokens,
        title="New Update",
        body="A new feature has been released!"
    )

    return result
```

These endpoints allow testing notifications via API.

---

# 3. Redis Queue Configuration

Redis is used as a **message broker** between the scheduler and worker.

### File

```
app/core/redis_client.py
```

### Code

```python
import redis.asyncio as redis

redis_client = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)

REDIS_NOTIFICATION_QUEUE = "notification_queue"
```

The queue name used is:

```
notification_queue
```

---

# 4. APScheduler Background Jobs

APScheduler is used to trigger **scheduled notification events**.

Examples:

* Birthday notifications
* Anniversary notifications
* Follow-up reminders

### File

```
app/core/scheduler.py
```

### Code

```python
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from apscheduler.triggers.cron import CronTrigger
from apscheduler.triggers.interval import IntervalTrigger

from app.db.session import AsyncSessionLocal
from app.services.notification import create_celebration_notifications, create_followup_notification

scheduler = AsyncIOScheduler()


async def run_celebration_job():
    print("Scheduler triggered celebration job")

    async with AsyncSessionLocal() as db:
        await create_celebration_notifications(db)


async def run_followup_job():
    async with AsyncSessionLocal() as db:
        await create_followup_notification(db)


def start_scheduler():

    print("Starting scheduler...")

    scheduler.add_job(
        run_celebration_job,
        IntervalTrigger(seconds=10),
        id="celebration_notification_job",
        replace_existing=True,
        max_instances=1,
        coalesce=True,
        misfire_grace_time=3600
    )

    scheduler.add_job(
        run_followup_job,
        CronTrigger(hour=0, minute=10),
        id="followup_notification_job",
        replace_existing=True,
        max_instances=1
    )

    scheduler.start()

    print("Jobs:", scheduler.get_jobs())


def stop_scheduler():
    print("Scheduler shutdown")
    scheduler.shutdown()
```

---

# 5. Scheduler Initialization in FastAPI

The scheduler starts during application startup.

### File

```
main.py
```

### Code

```python
@asynccontextmanager
async def lifespan(app: FastAPI):

    init_firebase()

    start_scheduler()

    yield

    stop_scheduler()
```

This ensures background jobs run while the application is active.

---

# 6. Redis Job Producer

When a notification is created, a job is pushed into Redis.

### File

```
app/services/notification_queue.py
```

### Code

```python
import json
from app.core.redis_client import redis_client, REDIS_NOTIFICATION_QUEUE


async def push_notification_job(username: str, title: str, message: str):

    payload = {
        "username": username,
        "title": title,
        "message": message,
    }

    await redis_client.lpush(
        REDIS_NOTIFICATION_QUEUE,
        json.dumps(payload)
    )
```

Example job payload:

```
{
  "username": "john_doe",
  "title": "Happy Birthday 🎂",
  "message": "Today is John's birthday"
}
```

---

# 7. Notification Worker

The worker consumes jobs from Redis and sends notifications via Firebase.

### File

```
app/workers/notification_worker.py
```

### Code

```python
import json
import asyncio

from app.core.redis_client import redis_client, REDIS_NOTIFICATION_QUEUE
from app.db.session import AsyncSessionLocal
from app.services.firebase_service import send_bulk_notification
from app.services.users import get_user_fcm_token_list
from app.core.firebase_config import init_firebase


async def notification_worker():

    print("🚀 Notification worker started...")

    init_firebase()

    while True:

        job = await redis_client.brpop(REDIS_NOTIFICATION_QUEUE)

        if not job:
            continue

        payload = json.loads(job[1])

        username = payload["username"]
        title = payload["title"]
        message = payload["message"]

        try:

            async with AsyncSessionLocal() as db:

                token_list = await get_user_fcm_token_list(db)

                if not token_list:
                    continue

                token_list = [str(token).strip() for token in token_list if token]

                await send_bulk_notification(
                    token_list,
                    title=title,
                    body=message
                )

                print(f"Push sent to {username}")

        except Exception as e:

            print(f"Push failed for {username}: {e}")


if __name__ == "__main__":
    asyncio.run(notification_worker())
```

---

# 8. Worker Execution

Run the worker separately:

```
python -m app.workers.notification_worker
```

This worker will continuously listen to the Redis queue.

---

# 9. Notification Flow

Complete system workflow:

```
Scheduler
   │
   ▼
Create Notification
   │
   ▼
Insert into DB
   │
   ▼
Push Redis Job
   │
   ▼
Redis Queue
   │
   ▼
Notification Worker
   │
   ▼
Fetch User FCM Tokens
   │
   ▼
Send Firebase Push Notification
   │
   ▼
Mobile Device
```

---

# 10. Benefits of This Architecture

### Asynchronous Processing

Notifications do not block API requests.

### Scalability

Multiple workers can consume jobs from Redis.

### Reliability

Jobs remain in Redis until processed.

### Decoupled System

API and notification processing are independent.

---

# 11. Production Improvements (Recommended)

For production environments consider:

* Redis retry queue
* Dead-letter queue
* Token cleanup service
* Worker autoscaling
* Batch Firebase sending

---

# 12. Example Job in Redis

Check jobs inside Redis:

```
LRANGE notification_queue 0 -1
```

Example output:

```
1) {"username":"john_doe","title":"Happy Birthday 🎂","message":"Today is John's birthday"}
```

---

# 13. Monitoring Logs

Worker logs:

```
🚀 Notification worker started
Processing job...
Push sent to john_doe
```

Scheduler logs:

```
Starting scheduler...
Scheduler triggered celebration job
```

---

# Conclusion

This architecture enables a **robust, scalable, and asynchronous notification delivery system** by combining:

* FastAPI
* APScheduler
* Redis Queue
* Worker Processes
* Firebase Cloud Messaging
