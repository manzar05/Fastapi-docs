### Async Database Configuration
> Create a file:
> app/db/database.py
```
from sqlalchemy.ext.asyncio import (
    create_async_engine,
    async_sessionmaker,
    AsyncSession
)

DATABASE_URL = "postgresql+asyncpg://user:password@localhost:5432/mydb"

engine = create_async_engine(
    DATABASE_URL,
    echo=True
)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False
)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```
### Create Base Model
> app/db/base.py
```
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

### Create Database Model
> app/models/user.py
```
from sqlalchemy import String, Integer
from sqlalchemy.orm import Mapped, mapped_column
from app.db.base import Base

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True,
        index=True
    )

    name: Mapped[str] = mapped_column(
        String(100),
        nullable=False
    )

    email: Mapped[str] = mapped_column(
        String(150),
        unique=True,
        nullable=False,
        index=True
    )
```

### Create Pydantic Schemas
```
from pydantic import BaseModel, EmailStr

class UserCreate(BaseModel):
    name: str
    email: EmailStr

class UserResponse(BaseModel):
    id: int
    name: str
    email: EmailStr

    class Config:
        from_attributes = True
```
> - Install email validator:
> - pip install pydantic[email]

### Create Tables
> Install Alembic by running this command: pip install alembic
> Initialize Alembic: alembic init alembic
> This creates:
```
alembic/
    versions/
alembic.ini
```
Here is sample Table Structure:
```
from sqlalchemy import Integer, String
from sqlalchemy.orm import Mapped, mapped_column

from app.db.base import Base

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String)
```
> Configure Alembic

### Create First Migration
```
alembic revision --autogenerate -m "create users table"
```

### Apply Migration
```
alembic upgrade head
```

### Create Async CRUD APIs
```
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

from app.db.database import get_db
from app.models.user import User
from app.schemas.user import UserCreate, UserResponse

router = APIRouter(
    prefix="/users",
    tags=["Users"]
)
```

### Create User API
```
@router.post("/", response_model=UserResponse)
async def create_user(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    user = User(
        name=user_data.name,
        email=user_data.email
    )

    db.add(user)
    await db.commit()
    await db.refresh(user)

    return user
```
### Get All Users API
```
@router.get("/", response_model=list[UserResponse])
async def get_users(
    db: AsyncSession = Depends(get_db)
):
    result = await db.execute(select(User))
    users = result.scalars().all()

    return users
```
### Get Single User API
```
@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db)
):
    result = await db.execute(
        select(User).where(User.id == user_id)
    )

    user = result.scalar_one_or_none()

    if not user:
        raise HTTPException(
            status_code=404,
            detail="User not found"
        )

    return user
```
### Update User API
```
@router.put("/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: int,
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    result = await db.execute(
        select(User).where(User.id == user_id)
    )

    user = result.scalar_one_or_none()

    if not user:
        raise HTTPException(
            status_code=404,
            detail="User not found"
        )

    user.name = user_data.name
    user.email = user_data.email

    await db.commit()
    await db.refresh(user)

    return user
```
### Delete User API
```
@router.delete("/{user_id}")
async def delete_user(
    user_id: int,
    db: AsyncSession = Depends(get_db)
):
    result = await db.execute(
        select(User).where(User.id == user_id)
    )

    user = result.scalar_one_or_none()

    if not user:
        raise HTTPException(
            status_code=404,
            detail="User not found"
        )

    await db.delete(user)
    await db.commit()

    return {
        "message": "User deleted successfully"
    }
```

### Include Router in Main App
```
from fastapi import FastAPI
from app.routes import user

app = FastAPI()

app.include_router(user.router)
```
