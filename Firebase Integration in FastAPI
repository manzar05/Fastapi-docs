# Firebase Push Notification Integration

This document explains how **Firebase Cloud Messaging (FCM)** is integrated in the application to send **single-user** and **bulk push notifications**.

---

# 1. Firebase Initialization

Firebase must be initialized **once per process** before sending notifications.

### File Location

```
app/core/firebase_config.py
```

### Code

```python
import firebase_admin
from firebase_admin import credentials

def init_firebase():
    if not firebase_admin._apps:
        cred = credentials.Certificate("app/firebase.json")
        firebase_admin.initialize_app(cred)
```

### Why this check is needed

```python
if not firebase_admin._apps:
```

This prevents the error:

```
ValueError: The default Firebase app already exists
```

because Firebase can only be initialized **once per process**.

---

# 2. Firebase Initialization in FastAPI

Firebase is initialized during application startup.

### File

```
main.py
```

### Code

```python
from app.core.firebase_config import init_firebase

init_firebase()
```

This ensures Firebase is available across all services that need push notifications.

---

# 3. Sending Push Notification to a Single User

### Function

```
send_push_notification
```

### Code

```python
from firebase_admin import messaging

async def send_push_notification(token: str, title: str, body: str):

    message = messaging.Message(
        notification=messaging.Notification(
            title=title,
            body=body
        ),
        token=token.strip()
    )

    try:
        response = messaging.send(message)
        return {"success": True, "message_id": response}

    except messaging.UnregisteredError:
        print("Token is invalid. Remove it from DB.")
        return {"success": False, "reason": "Token unregistered"}
```

### Example Usage

```python
token = await get_user_fcm_token(db, current_user.username)

message = await send_push_notification(
    token,
    title="[Single] Route Logs Saved",
    body="[Single] Your realtime logs have been created"
)

print("Firebase message:", message)
```

---

# 4. Sending Push Notifications to Multiple Users

### Function

```
send_bulk_notification
```

### Code

```python
from firebase_admin import messaging

async def send_bulk_notification(tokens: list[str], title: str, body: str):

    tokens_list = [t.strip() for t in tokens if t]

    if not tokens_list:
        return {"success_count": 0, "failure_count": 0}

    message = messaging.MulticastMessage(
        notification=messaging.Notification(
            title=title,
            body=f"[This Bulk Notification Shared]: {body}"
        ),
        tokens=tokens_list
    )

    response = messaging.send_each_for_multicast(message)

    invalid_tokens = []

    for idx, resp in enumerate(response.responses):
        if not resp.success:
            error = resp.exception
            print(f"Token failed: {tokens_list[idx]} -> {error}")

            if isinstance(error, messaging.UnregisteredError):
                invalid_tokens.append(tokens_list[idx])

    if invalid_tokens:
        print("Invalid tokens:", invalid_tokens)

    return {
        "success_count": response.success_count,
        "failure_count": response.failure_count,
        "invalid_tokens": invalid_tokens
    }
```

---

# 5. Example: Bulk Notification Test

```python
token_list = [
    "fPpSDlVNQMaE2xUWnRCBqG:APA91...",
    "fdVk3D98TRqmetu034fTHv:APA91..."
]

message = await send_bulk_notification(
    token_list,
    title="Congratulation! You did it",
    body="This Notification is for Testing Purpose"
)

print("Firebase message:", message)
```

---

# 6. Token Storage Strategy

Each user stores an **FCM token** in the database.

Example field:

```
User.fcm_token
```

Tokens can be retrieved using SQL queries:

```python
select(User.fcm_token)
```

Bulk notifications can also target users based on roles or designations.

Example:

```python
User.designation == 4
```

---

# 7. Handling Invalid Tokens

If Firebase returns:

```
UnregisteredError
```

the token should be removed from the database.

Example:

```python
if isinstance(error, messaging.UnregisteredError):
    invalid_tokens.append(tokens_list[idx])
```

Later this list can be used to clean invalid tokens.

---

# 8. Notification Flow

The complete notification pipeline works as follows:

```
Scheduler (APScheduler)
        ↓
Create Notification Event
        ↓
Store Notification in Database
        ↓
Push Job to Redis Queue
        ↓
Worker Processes Queue
        ↓
Fetch FCM Tokens
        ↓
Send Firebase Push Notification
```

---

# 9. Firebase Service Account File

Firebase requires a service account file.

Location used in this project:

```
app/firebase.json
```

This file is generated from:

```
Firebase Console
→ Project Settings
→ Service Accounts
→ Generate Private Key
```

⚠️ **Important:** Never commit this file to public repositories.

Add to `.gitignore`:

```
app/firebase.json
```

---

# 10. Firebase Limits

| Feature                      | Limit               |
| ---------------------------- | ------------------- |
| Multicast tokens per request | 500                 |
| Single message payload       | 4 KB                |
| Rate limits                  | Managed by Firebase |

If sending to many users, send notifications in **batches of 500 tokens**.

---

# 11. Best Practices

* Initialize Firebase only once per process
* Strip whitespace from tokens
* Remove invalid tokens from the database
* Use batch notifications for performance
* Secure the Firebase service account JSON file
