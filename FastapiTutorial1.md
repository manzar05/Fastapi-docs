## FastAPI Advanced Features Documentation
---
### 1. Request Parameters from Multiple Sources
> FastAPI allows you to declare request data from different parts of an HTTP request:
> - Path Parameters
> - Query Parameters
> - Headers
> - Cookies
> - Form Fields
> - File Uploads
---
### Path Parameters
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id}
```
### Query Parameters
```
@app.get("/search")
async def search(q: str, page: int = 1):
    return {
        "query": q,
        "page": page
    }
```
> Request:
> ```
> GET /search?q=fastapi&page=2
> ```

### Header Parameters
```
from fastapi import Header

@app.get("/headers")
async def read_headers(user_agent: str = Header()):
    return {"User-Agent": user_agent}
```
> Custom headers:
> ```
> @app.get("/custom-header")
async def custom_header(x_token: str = Header()):
    return {"X-Token": x_token}
> ```

### Cookie Parameters
```
from fastapi import Cookie

@app.get("/cookies")
async def read_cookie(session_id: str = Cookie(default=None)):
    return {"session_id": session_id}
```

### Form Fields
Used when receiving HTML form data.
```
from fastapi import Form

@app.post("/login")
async def login(
    username: str = Form(...),
    password: str = Form(...)
):
    return {
        "username": username
    }
```
> Install dependency:
> pip install python-multipart

### File Uploads
> Single File Upload.
```
from fastapi import UploadFile, File

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    return {
        "filename": file.filename,
        "content_type": file.content_type
    }
```
> ... is called the Ellipsis object.
> The request must contain a file.
> If you want the file to be optional:
```from typing import Optional
from fastapi import UploadFile, File

@app.post("/upload")
async def upload_file(file: Optional[UploadFile] = None):
    return {"file": file}
```
OR
```
file: UploadFile | None = File(None)
```

> Multiple Files Upload.
```
from fastapi import UploadFile, File

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    return {
        "filename": file.filename,
        "content_type": file.content_type
    }
```
---

### 2. Validation Constraints
FastAPI uses Pydantic validation internally.
#### String Validation
```
from fastapi import Query

@app.get("/items")
async def get_items(
    name: str = Query(
        ...,
        min_length=3,
        max_length=50,
        regex="^[a-zA-Z0-9_]+$"
    )
):
    return {"name": name}
```
> Validation features:
> - min_length
> - max_length
> - regex
> - gt (greater than)
> - ge (greater or equal)
> - lt
> - le

#### Numeric Validation
```
@app.get("/products")
async def products(
    price: float = Query(..., gt=0, le=10000)
):
    return {"price": price}
```

#### Pydantic Model Validation
```
from pydantic import BaseModel, Field

class User(BaseModel):
    username: str = Field(
        ...,
        min_length=3,
        max_length=20
    )

    age: int = Field(
        ...,
        ge=18,
        le=100
    )
```
---
### 3. Dependency Injection System
FastAPI provides a powerful dependency injection mechanism using Depends.
#### Basic Dependency
```
from fastapi import Depends

def common_parameters(q: str = None):
    return {"q": q}

@app.get("/items")
async def read_items(
    commons: dict = Depends(common_parameters)
):
    return commons
```
#### Database Dependency Example
```
from sqlalchemy.orm import Session

def get_db():
    db = Session()
    try:
        yield db
    finally:
        db.close()
```
> Usage:
> ```
> @app.get("/users")
async def users(db: Session = Depends(get_db)):
    return {"message": "DB connected"}
> ```

#### Dependency Classes
```
class Pagination:
    def __init__(self, limit: int = 10, skip: int = 0):
        self.limit = limit
        self.skip = skip

@app.get("/posts")
async def posts(
    pagination: Pagination = Depends()
):
    return pagination.__dict__
```
#### Global Dependencies
```
app = FastAPI(
    dependencies=[Depends(common_parameters)]
)
```
---

### 4. Security and Authentication
FastAPI supports several authentication mechanisms.
#### OAuth2 with JWT Tokens
OAuth2 Password Flow
```
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token"
)
```
#### Create Access Token
```
from jose import jwt
from datetime import datetime, timedelta

SECRET_KEY = "secret"
ALGORITHM = "HS256"

def create_access_token(data: dict):
    to_encode = data.copy()

    expire = datetime.utcnow() + timedelta(minutes=30)

    to_encode.update({"exp": expire})

    encoded_jwt = jwt.encode(
        to_encode,
        SECRET_KEY,
        algorithm=ALGORITHM
    )

    return encoded_jwt
```
#### Protected Routes
```
@app.get("/protected")
async def protected_route(
    token: str = Depends(oauth2_scheme)
):
    return {"token": token}
```
> Install JWT Dependencies
> pip install python-jose[cryptography] passlib[bcrypt]

---

### 5. Deeply Nested JSON Models with Pydantic
FastAPI handles nested data structures naturally.
#### Nested Models Example
```
from typing import List
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    country: str

class User(BaseModel):
    name: str
    address: Address

class Company(BaseModel):
    company_name: str
    employees: List[User]
```
#### Example Request Body
```
{
  "company_name": "OpenAI",
  "employees": [
    {
      "name": "John",
      "address": {
        "city": "Delhi",
        "country": "India"
      }
    }
  ]
}
```

    
