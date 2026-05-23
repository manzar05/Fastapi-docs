### Required Packages
```
pip install fastapi uvicorn sqlalchemy asyncpg alembic pydantic[email]
```
> Import Required Things
```
from sqlalchemy import select, update, delete, func
from sqlalchemy.ext.asyncio import AsyncSession
from fastapi import Depends, HTTPException, APIRouter

from app.db.database import get_db
from app.models.user import User
from app.schemas.user import UserCreate, UserUpdate, UserResponse
```
### Create Record
```
  # Create a row with name and email in User table
  user = User(
      name=user_data.name,
      email=user_data.email
  )

  db.add(user)
  # This line will reflect changes in db
  await db.commit()
  # This line will return new inserted row from db
  await db.refresh(user)

  return user
```

### Get All Records
```
query = select(User)
result = await db.execute(query)
users = result.scalars().all()
return users
```

### Get Single Record by ID
```
query = select(User).where(User.id == user_id)
result = await db.execute(query)
user = result.scalar_one_or_none()
if not user:
    raise HTTPException(
        status_code=404,
        detail="User not found"
    )
return user
```

### Filter Records
```
query = select(User).where(User.is_active == True)
result = await db.execute(query)
users = result.scalars().all()
return users
```

### Search Records
```
query = select(User).where(
    User.name.ilike(f"%{keyword}%")
)
result = await db.execute(query)
users = result.scalars().all()
return users
```
### Multiple Conditions (AND Condition)
```
query = select(User).where(
    User.name.ilike(f"%{keyword}%"),
    User.is_active == is_active
)
result = await db.execute(query)
users = result.scalars().all()
return users
```

### OR Condition
```
query = select(User).where(
    or_(
        User.name.ilike(f"%{keyword}%"),
        User.email.ilike(f"%{keyword}%")
    )
)
result = await db.execute(query)
users = result.scalars().all()
return users
```

### Ordering
```
query = select(User).order_by(User.id.desc())
result = await db.execute(query)
users = result.scalars().all()
return users
```
### Count Records
```
query = select(func.count(User.id))
result = await db.execute(query)
total = result.scalar()
return {
    "total_users": total
}
```
---

### Direct Update Query
```
query = (
    update(User)
    .where(User.id == user_id)
    .values(is_active=is_active)
)
await db.execute(query)
await db.commit()
return {
    "message": "User status updated"
}
```
### Select Specific Columns
```
query = select(User.id, User.email)
result = await db.execute(query)
users = result.all()
return [
    {
        "id": user.id,
        "email": user.email
    }
    for user in users
]
```
### Rollback on Error
```
@router.post("/safe-create")
async def safe_create_user(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    try:
        user = User(
            name=user_data.name,
            email=user_data.email
        )

        db.add(user)

        await db.commit()
        await db.refresh(user)

        return user

    except Exception as e:
        await db.rollback()

        raise HTTPException(
            status_code=500,
            detail=str(e)
        )
```

| Code | Description |
|---|---|
| `db.add(obj)` | Add new object |
| `await db.commit()` | Save changes |
| `await db.refresh(obj)` | Reload object from DB |
| `await db.execute(query)` | Execute query |
| `result.scalars().all()` | Get list of ORM objects |
| `result.scalar_one_or_none()` | Get one object or `None` |
| `await db.delete(obj)` | Delete object |
| `await db.rollback()` | Rollback on error |
