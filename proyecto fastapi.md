# Curso Práctico FastAPI desde Cero

## Proyecto Completo con Arquitectura Profesional

Este ejemplo está diseñado para estudiantes de Programación que desean aprender FastAPI utilizando una estructura similar a proyectos empresariales y microservicios.

---

# Objetivo del proyecto

Construir una API REST con:

* FastAPI
* SQLite
* SQLModel
* CRUD de Usuarios
* CRUD de Productos (Items)
* Validaciones
* Dependencias
* Servicios externos simulados
* Pruebas unitarias

---

# Paso 1. Crear proyecto

Abrir VS Code.

Crear carpeta:

```text
fastapi_microservice
```

Entrar a la carpeta:

```bash
cd fastapi_microservice
```

---

# Paso 2. Crear entorno virtual

```bash
python -m venv .venv
```

Activar:

Windows

```bash
.venv\Scripts\activate
```

Linux/Mac

```bash
source .venv/bin/activate
```

---

# Paso 3. Instalar dependencias

```bash
pip install fastapi uvicorn sqlmodel pytest
```

Guardar dependencias:

```bash
pip freeze > requirements.txt
```

---

# Paso 4. Crear estructura

```text
fastapi_microservice
│
├── app
│   ├── __init__.py
│   ├── main.py
│   ├── dependencies.py
│   │
│   ├── routers
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── items.py
│   │
│   ├── crud
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   │
│   ├── schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   │
│   ├── models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── item.py
│   │
│   ├── external_services
│   │   ├── __init__.py
│   │   ├── email.py
│   │   └── notification.py
│   │
│   └── utils
│       ├── __init__.py
│       ├── authentication.py
│       └── validation.py
│
├── tests
│   ├── __init__.py
│   ├── test_main.py
│   ├── test_users.py
│   └── test_items.py
│
├── requirements.txt
├── README.md
└── .gitignore
```

---

# Paso 5. Crear Base de Datos

## app/dependencies.py

```python
from sqlmodel import SQLModel
from sqlmodel import Session
from sqlmodel import create_engine

DATABASE_URL = "sqlite:///database.db"

engine = create_engine(
    DATABASE_URL,
    echo=True
)

def create_db():
    SQLModel.metadata.create_all(engine)

def get_session():
    with Session(engine) as session:
        yield session
```

---

# Paso 6. Crear modelos

## app/models/user.py

```python
from sqlmodel import SQLModel, Field

class User(SQLModel, table=True):

    id: int | None = Field(
        default=None,
        primary_key=True
    )

    name: str
    email: str
```

---

## app/models/item.py

```python
from sqlmodel import SQLModel, Field

class Item(SQLModel, table=True):

    id: int | None = Field(
        default=None,
        primary_key=True
    )

    name: str
    price: float
```

---

# Paso 7. Crear Schemas

Los schemas validan la entrada y salida.

---

## app/schemas/user.py

```python
from pydantic import BaseModel

class UserCreate(BaseModel):
    name: str
    email: str

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
```

---

## app/schemas/item.py

```python
from pydantic import BaseModel

class ItemCreate(BaseModel):
    name: str
    price: float

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float
```

---

# Paso 8. CRUD Usuarios

## app/crud/user.py

```python
from sqlmodel import Session
from sqlmodel import select

from app.models.user import User

def create_user(
    session: Session,
    user_data
):
    user = User(**user_data.model_dump())

    session.add(user)
    session.commit()
    session.refresh(user)

    return user

def get_users(session: Session):

    statement = select(User)

    return session.exec(statement).all()
```

---

# Paso 9. CRUD Items

## app/crud/item.py

```python
from sqlmodel import Session
from sqlmodel import select

from app.models.item import Item

def create_item(
    session: Session,
    item_data
):

    item = Item(**item_data.model_dump())

    session.add(item)
    session.commit()
    session.refresh(item)

    return item

def get_items(session: Session):

    statement = select(Item)

    return session.exec(statement).all()
```

---

# Paso 10. Servicio de Email

## app/external_services/email.py

```python
def send_email(email):

    print(
        f"Correo enviado a {email}"
    )
```

---

# Paso 11. Servicio de Notificaciones

## app/external_services/notification.py

```python
def send_notification(message):

    print(
        f"Notificación: {message}"
    )
```

---

# Paso 12. Utilidades

## app/utils/authentication.py

```python
def verify_token():

    return True
```

---

## app/utils/validation.py

```python
def validate_email(email):

    return "@" in email
```

---

# Paso 13. Router Usuarios

## app/routers/users.py

```python
from fastapi import APIRouter
from fastapi import Depends

from sqlmodel import Session

from app.dependencies import get_session

from app.schemas.user import UserCreate

from app.crud.user import (
    create_user,
    get_users
)

from app.external_services.email import send_email

router = APIRouter(
    prefix="/users",
    tags=["Users"]
)

@router.post("/")
def create_new_user(
    user: UserCreate,
    session: Session = Depends(get_session)
):

    new_user = create_user(
        session,
        user
    )

    send_email(user.email)

    return new_user

@router.get("/")
def list_users(
    session: Session = Depends(get_session)
):

    return get_users(session)
```

---

# Paso 14. Router Items

## app/routers/items.py

```python
from fastapi import APIRouter
from fastapi import Depends

from sqlmodel import Session

from app.dependencies import get_session

from app.schemas.item import ItemCreate

from app.crud.item import (
    create_item,
    get_items
)

router = APIRouter(
    prefix="/items",
    tags=["Items"]
)

@router.post("/")
def create_new_item(
    item: ItemCreate,
    session: Session = Depends(get_session)
):

    return create_item(
        session,
        item
    )

@router.get("/")
def list_items(
    session: Session = Depends(get_session)
):

    return get_items(session)
```

---

# Paso 15. Archivo principal

## app/main.py

```python
from fastapi import FastAPI

from app.dependencies import create_db

from app.routers.users import router as user_router
from app.routers.items import router as item_router

app = FastAPI(
    title="Microservice API",
    version="1.0"
)

@app.on_event("startup")
def startup():

    create_db()

app.include_router(user_router)
app.include_router(item_router)

@app.get("/")
def home():

    return {
        "message": "API funcionando"
    }
```

---

# Paso 16. Ejecutar servidor

Desde la raíz:

```bash
uvicorn app.main:app --reload
```

---

# Paso 17. Abrir Swagger

FastAPI genera documentación automática.

Swagger:

```text
http://127.0.0.1:8000/docs
```

ReDoc:

```text
http://127.0.0.1:8000/redoc
```

---

# Paso 18. Probar Usuarios

POST

```json
{
  "name": "Juan",
  "email": "juan@gmail.com"
}
```

Endpoint:

```text
POST /users/
```

---

GET

```text
GET /users/
```

Resultado:

```json
[
  {
    "id":1,
    "name":"Juan",
    "email":"juan@gmail.com"
  }
]
```

---

# Paso 19. Probar Items

POST

```json
{
  "name":"Laptop",
  "price":15000
}
```

Endpoint:

```text
POST /items/
```

---

GET

```text
GET /items/
```

Resultado:

```json
[
  {
    "id":1,
    "name":"Laptop",
    "price":15000
  }
]
```

---

# Paso 20. Crear pruebas

## tests/test_main.py

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)

def test_home():

    response = client.get("/")

    assert response.status_code == 200
```

---

## tests/test_users.py

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)

def test_users():

    response = client.get("/users/")

    assert response.status_code == 200
```

---

## tests/test_items.py

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)

def test_items():

    response = client.get("/items/")

    assert response.status_code == 200
```

---

# Paso 21. Ejecutar pruebas

```bash
pytest
```

---

# Resultado Final

Al finalizar tendrás una API profesional con:

✅ Arquitectura por capas

✅ Routers

✅ CRUD

✅ SQLite

✅ SQLModel

✅ Schemas Pydantic

✅ Dependencias (Depends)

✅ Servicios externos

✅ Utilidades

✅ Testing con Pytest

✅ Documentación Swagger

✅ Base sólida para evolucionar a JWT, OAuth2, PostgreSQL, Docker, Firebase o microservicios empresariales.

Como siguiente práctica, recomiendo ampliar este proyecto a un **CRUD completo (Create, Read, Update, Delete) para Usuarios e Items**, agregando endpoints `GET por ID`, `PUT` y `DELETE`, manejo de errores HTTP y relaciones entre tablas. Esto acerca mucho más el proyecto a un entorno real de desarrollo backend.
