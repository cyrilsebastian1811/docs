# __:simple-fastapi:{.lg .top} FastAPI__

__Layers:__

- Entity (ORM models)
- DTOs
- Validation (DTO + entity)
- DAO / Repository
- Service layer
- Controller (API routes)

```text title="Project Structure"
app/
 ├── main.py
 ├── database/
 │    ├── session.py
 │    └── base.py
 │
 ├── models/        # Entities
 │    └── user.py
 │
 ├── schemas/       # DTOs
 │    └── user_dto.py
 │
 ├── repositories/  # DAO / Repository layer
 │    └── user_repository.py
 │
 ├── services/
 │    └── user_service.py
 │
 ├── controllers/
 │    └── user_controller.py
 │
 └── validators/
      └── user_validator.py
```

```text title="Typical libraries"
fastapi
uvicorn
sqlalchemy
pydantic
```

## Database Setup

Creates the DB session.

=== "SQLAlchemy approach"

    ```python title="database/session.py"
    from sqlalchemy import create_engine
    from sqlalchemy.orm import sessionmaker

    DATABASE_URL = "postgresql://user:password@localhost/db"

    engine = create_engine(DATABASE_URL, echo=True)
    SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
    ```

    ```python title="database/base.py"
    from sqlalchemy.orm import declarative_base

    Base = declarative_base()
    ```

    ??? note "`declarative_base()`"

        Creates a base class that your database model classes will inherit from. This base class contains the machinery SQLAlchemy needs to:
        - Map Python classes to database tables
        - Track table metadata
        - Register models with the ORM

        `Base` acts as:
        - A registry of all ORM models
        - A container for metadata used to create tables

        ```python
        # This tells SQLAlchemy to create all tables defined by classes that inherit from Base.
        Base.metadata.create_all(engine)
        ```

=== "SQLAlchemy approach"

    Built on top of SQLAlchemy + Pydantic

    ```python
    from sqlmodel import Field, Session, SQLModel, create_engine, select

    sqlite_file_name = "database.db"
    sqlite_url = f"sqlite:///{sqlite_file_name}"

    connect_args = {"check_same_thread": False}
    engine = create_engine(sqlite_url, connect_args=connect_args)

    def get_session():
        with Session(engine) as session:
            yield session


    SessionDep = Annotated[Session, Depends(get_session)]
    ```


## Entity (ORM Model) - SQLAlchemy

Entities represent database tables.

=== "One-To-One Mapping"

    === "Unidirectional"

        ```python
        from sqlalchemy import Column, Integer, ForeignKey
        from sqlalchemy.orm import relationship

        class Profile(Base):
            __tablename__ = "profiles"
            id = Column(Integer, primary_key=True)

        class User(Base):
            __tablename__ = "users"
            id = Column(Integer, primary_key=True)
            profile_id = Column(Integer, ForeignKey("profiles.id"), unique=True)

            profile = relationship("Profile", uselist=False)
        ```

    === "Bidirectional"

        ```python
        from sqlalchemy import Column, Integer, ForeignKey
        from sqlalchemy.orm import relationship

        class User(Base):
            __tablename__ = "users"
            id = Column(Integer, primary_key=True)

            profile = relationship("Profile", back_populates="user", uselist=False)

        class Profile(Base):
            __tablename__ = "profiles"
            id = Column(Integer, primary_key=True)
            user_id = Column(Integer, ForeignKey("users.id"), unique=True)

            user = relationship("User", back_populates="profile")
        ```

        Here the owning side(i.e. Profile) contains ForeignKey. SQLAlchemy interprets that side as many-to-one, which already returns a single object. Thus, no need for `user = relationship("User", back_populates="profile")`

    ==`unique=True` enforces one-to-one==

=== "One-To-Many Mapping"

    === "Unidirectional"

        ```python
        from sqlalchemy import Column, Integer, ForeignKey
        from sqlalchemy.orm import relationship

        class User(Base):
            __tablename__ = "users"
            id = Column(Integer, primary_key=True)

            orders = relationship("Order")

        class Order(Base):
            __tablename__ = "orders"
            id = Column(Integer, primary_key=True)
            user_id = Column(Integer, ForeignKey("users.id"))
        ```

    === "Bidirectional"

        ```python
        from sqlalchemy import Column, Integer, ForeignKey
        from sqlalchemy.orm import relationship

        class User(Base):
            __tablename__ = "users"
            id = Column(Integer, primary_key=True)

            orders = relationship("Order", back_populates="user")

        class Order(Base):
            __tablename__ = "orders"
            id = Column(Integer, primary_key=True)
            user_id = Column(Integer, ForeignKey("users.id"))

            user = relationship("User", back_populates="orders")
        ```

=== "Many-To-Many Mapping"

    Requires an association table.

    === "Unidirectional"

        ```python
        from sqlalchemy import Column, Integer, ForeignKey
        from sqlalchemy.orm import relationship
        from sqlalchemy import Table

        student_course = Table(
            "student_course",
            Base.metadata,
            Column("student_id", ForeignKey("students.id"), primary_key=True),
            Column("course_id", ForeignKey("courses.id"), primary_key=True),
        )

        class Student(Base):
            __tablename__ = "students"
            id = Column(Integer, primary_key=True)

            courses = relationship("Course", secondary=student_course)

        class Course(Base):
            __tablename__ = "courses"
            id = Column(Integer, primary_key=True)
        ```

    === "Bidirectional"

        ```python
        from sqlalchemy import Column, Integer, ForeignKey
        from sqlalchemy.orm import relationship
        from sqlalchemy import Table

        student_course = Table(
            "student_course",
            Base.metadata,
            Column("student_id", ForeignKey("students.id"), primary_key=True),
            Column("course_id", ForeignKey("courses.id"), primary_key=True),
        )

        class Student(Base):
            __tablename__ = "students"
            id = Column(Integer, primary_key=True)

            courses = relationship(
                "Course",
                secondary=student_course,
                back_populates="students"
            )

        class Course(Base):
            __tablename__ = "courses"
            id = Column(Integer, primary_key=True)

            students = relationship(
                "Student",
                secondary=student_course,
                back_populates="courses"
            )
        ```


## DTO (Data Transfer Objects)

DTOs define what enters and leaves the API. Validation happens automatically in DTOs.

```python title="schemas/user_dto.py"
from pydantic import BaseModel, EmailStr, Field

class UserCreateDTO(BaseModel):
    name: str = Field(min_length=2, max_length=100)
    email: EmailStr
    age: int = Field(gt=0, lt=120)

class UserResponseDTO(BaseModel):
    id: int
    name: str
    email: str
    age: int

    class Config:
        from_attributes = True
```

```python title=""
# DTO being used
@app.post("/users", response_model=UserResponseDTO)
def create_user(user: UserCreateDTO, db: Session):
    new_user = User(
        name=user.name,
        email=user.email,
        age=user.age
    )
    db.add(new_user)
    db.commit()
    db.refresh(new_user)

    return new_user
```

??? note "`from_attributes`"

    Without `from_attributes`, Pydantic expects a dictionary.

    ```python
    user_dict = {
        "id": 1,
        "name": "Alice",
        "email": "alice@test.com",
        "age": 30
    }

    UserResponseDTO(**user_dict)  # works
    ```

    With `from_attributes = True`, Now Pydantic reads attributes from objects

    ```python
    user = db.query(User).first()

    UserResponseDTO.model_validate(user)
    ```


## Repository Layer (DAO)

__Responsibilities:__

- CRUD
- Query optimization
- No business logic

```python title="repositories/user_repository.py"
from sqlalchemy.orm import Session
from app.models.user import User

class UserRepository:

    def __init__(self, db: Session):
        self.db = db

    def create(self, user: User):
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        return user

    def get_by_id(self, user_id: int):
        return self.db.query(User).filter(User.id == user_id).first()

    def get_by_email(self, email: str):
        return self.db.query(User).filter(User.email == email).first()

    def list_users(self):
        return self.db.query(User).all()

    def delete(self, user: User):
        self.db.delete(user)
        self.db.commit()
```


## Service Layer

__Responsibilities:__

- Business rules
- Validation
- Orchestrating multiple repositories
- Transactions

```python title="services/user_service.py"
from app.models.user import User
from app.repositories.user_repository import UserRepository
from app.validators.user_validator import validate_user_entity

class UserService:

    def __init__(self, repository: UserRepository):
        self.repository = repository

    def create_user(self, dto):
        existing_user = self.repository.get_by_email(dto.email)
        if existing_user:
            raise Exception("User already exists")

        user = User(
            name=dto.name,
            email=dto.email,
            age=dto.age
        )

        validate_user_entity(user)

        return self.repository.create(user)

    def get_user(self, user_id):
        user = self.repository.get_by_id(user_id)
        if not user:
            raise Exception("User not found")
        return user

    def get_all_users(self):
        return self.repository.list_users()
```

## Dependency Injection

FastAPI supports DI out of the box.

```python
from app.database.session import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

from fastapi import Depends

@app.get("/users")
def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

```text
Incoming request
   ↓
Call dependency get_db()
   ↓
Provide db session to endpoint
   ↓
Run endpoint
   ↓
After response → close db session
```

==Because of `yield`, FastAPI treats it like a context-managed dependency.==

??? note "Function dependencies"

    ```python
    def get_token():
        return "abc123"

    @app.get("/")
    def home(token: str = Depends(get_token)):
        return token
    ```

??? note "Class dependencies"

    ```python
    class UserService:
        def __init__(self, db: Session):
            self.db = db

    def get_user_service(db: Session = Depends(get_db)):
        return UserService(db)

    @app.get("/users")
    def get_users(service: UserService = Depends(get_user_service)):
        return service.list_users()
    ```

## Controller Layer (API Routes)

Controller Layer (API Routes)

```python title="controllers/user_controller.py"
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from app.schemas.user_dto import UserCreateDTO, UserResponseDTO
from app.services.user_service import UserService
from app.repositories.user_repository import UserRepository
from app.database.session import SessionLocal
from app.dependencies import get_db

router = APIRouter(prefix="/users", tags=["Users"])

def get_user_service(db: Session = Depends(get_db)):
    repo = UserRepository(db)
    return UserService(repo)

@router.post("/", response_model=UserResponseDTO)
def create_user(
    dto: UserCreateDTO,
    service: UserService = Depends(get_user_service)
):
    return service.create_user(dto)

@router.get("/{user_id}", response_model=UserResponseDTO)
def get_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
):
    return service.get_user(user_id)

# /users?age=25&limit=10
@router.get("/", response_model=list[UserResponseDTO])
def list_users(
    age: int | None,
    limit: int = 20,
    service: UserService = Depends(get_user_service)
):
    return service.get_all_users(age, limit)
```

## Application Entry Point

```python
from fastapi import FastAPI
from app.controllers import user_controller
from app.database.base import Base
from app.database.session import engine

# SQLAlchemy approach
Base.metadata.create_all(bind=engine)

# # SQLModel approach
# def create_db_and_tables():
#     SQLModel.metadata.create_all(engine)

app = FastAPI()

app.include_router(user_controller.router)
```


## Asynchronous

FastAPI is built on:

- ASGI (Asynchronous Server Gateway Interface)
- Starlette
- asyncio

This means FastAPI can run both:

- synchronous endpoints
- asynchronous endpoints

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/sync")
def sync_route():
    return {"message": "sync response"}

@app.get("/async")
async def async_route():
    await asyncio.sleep(1)
    return {"message": "async response"}
```

### aiohttp

`aiohttp` is an async HTTP client and server library built directly on `asyncio`.

It is commonly used in FastAPI apps to:
- call external APIs
- handle async HTTP requests
- stream data

Why aiohttp instead of requests?

| Feature     | requests     | aiohttp      |
| ----------- | ------------ | ------------ |
| Async       | ❌            | ✅            |
| Streaming   | Limited      | Excellent    |
| Performance | Blocking     | Non-blocking |
| Concurrency | Thread based | Event loop   |

```python
from fastapi import FastAPI
import aiohttp

app = FastAPI()

@app.get("/weather")
async def weather():
    async with aiohttp.ClientSession() as session:
        async with session.get("https://api.weather.com/data") as r:
            data = await r.json()
    return data
```

__Benefits:__

- No thread blocking
- Thousands of concurrent calls possible

### SSE(Server-sent events)

SSE is a one-way streaming protocol where the server pushes updates to the client over HTTP.

```text
Browser
   ↓
EventSource API
   ↓
FastAPI SSE endpoint
   ↓
Async generator
   ↓
Event stream
```


```python title="server"
# pip install sse-starlette
import aiohttp
import asyncio
from fastapi import FastAPI
from sse_starlette.sse import EventSourceResponse

app = FastAPI()

async def stream_external_data():
    async with aiohttp.ClientSession() as session:
        while True:
            async with session.get("https://api.coindesk.com/v1/bpi/currentprice.json") as resp:
                data = await resp.json()
                price = data["bpi"]["USD"]["rate"]

            yield {"data": price}
            await asyncio.sleep(2)

@app.get("/bitcoin-stream")
async def bitcoin_stream():
    return EventSourceResponse(stream_external_data())
```

```js title="client"
const evtSource = new EventSource("/stream");

evtSource.onmessage = (event) => {
  console.log(event.data);
};
```

__SSE Uses:__

- HTTP connection (kept open)
- Chunked transfer encoding
- text/event-stream content type

```text title="Headers"
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
Transfer-Encoding: chunked
```

??? note "Event format"

    Each event is a set of field lines, followed by a blank line that marks the end of the event.

    ```text
    id: 42
    event: price-update
    retry: 3000
    data: {"symbol":"BTC","price":52000}
    ```

    === "`id`"

        - Unique identifier for the event.
        - Used for reconnection. Browser stores it automatically and sends `Last-Event-ID: 1` when reconnecting. This allows servers to resume streams.

    === "`event`"

        - Defines a custom event type.
        - If omitted defaults to `message`.

        ```python
        async def generator():
            while True:
                await asyncio.sleep(1)
                yield {
                    "event": "update",
                    "data": "price changed"
                }
        ```

        ```js
        evtSource.addEventListener("update", (event) => {
            console.log(event.data);
        });
        ```

    === "`data`"

        - The actual payload sent to the client.
        - If multiple data lines exist, the browser concatenates them with newline.

        ```text
        data: line 1
        data: line 2
        data: line 3
        ```

        ```text
        line 1
        line 2
        line 3
        ```

    === "`retry`"

        Tells the browser how long to wait before reconnecting.

        ```text
        retry: 5000
        ```





