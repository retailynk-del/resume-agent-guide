# Day 6: Auth Backend

> **Date:** Sprint 1, Day 6  
> **Focus:** User authentication backend (registration + login)  
> **Vertical Slice:** Users can register and login via API

---

## 🎯 Today's Goal

By end of today:
- ✅ User database model created (Dev 2)
- ✅ Registration endpoint working (Dev 1 - cross-training!)
- ✅ Login + JWT endpoint working (Dev 2)
- ✅ Auth middleware protecting routes (Dev 3 - learning backend!)
- ✅ Can test auth flow with curl/Postman

---

## 📋 Morning: Add Task to Jira

---

### Task AG-28: User Database Model & Connection

| Field | Value |
|-------|-------|
| **Task ID** | AG-28 |
| **Title** | User Database Model & Connection |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create Supabase database connection and User model using SQLAlchemy. This connects the backend to the Supabase database created on Day 1 and creates the foundation for auth.

**Acceptance Criteria:**
- [ ] SQLAlchemy connection setup in `backend/database/connection.py`
- [ ] User model created in `backend/database/models/user.py`
- [ ] Fields: id, email, password_hash, created_at
- [ ] Email is unique and indexed
- [ ] Connection tested successfully
- [ ] PR merged

**Note:** This uses the Supabase database created on Day 1, now connecting it to Python backend.

---

### Task AG-29: Registration Endpoint

| Field | Value |
|-------|-------|
| **Task ID** | AG-29 |
| **Title** | Registration Endpoint |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create POST /api/auth/register endpoint that hashes passwords and creates user accounts. Dev 1 gets backend API experience!

**Acceptance Criteria:**
- [ ] Endpoint: POST /api/auth/register
- [ ] Accepts: email, password
- [ ] Validates email format
- [ ] Hashes password with bcrypt
- [ ] Returns user_id on success
- [ ] Handles duplicate email error
- [ ] PR merged

---

### Task AG-30: Login & JWT Endpoint

| Field | Value |
|-------|-------|
| **Task ID** | AG-30 |
| **Title** | Login & JWT Endpoint |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create POST /api/auth/login endpoint that verifies credentials and returns JWT access tokens.

**Acceptance Criteria:**
- [ ] Endpoint: POST /api/auth/login
- [ ] Accepts: email, password
- [ ] Verifies password with bcrypt
- [ ] Generates JWT with secret key
- [ ] Returns access_token + user info
- [ ] Returns 401 for invalid credentials
- [ ] PR merged

---

### Task AG-31: Auth Middleware

| Field | Value |
|-------|-------|
| **Task ID** | AG-31 |
| **Title** | Auth Middleware |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create FastAPI dependency that validates JWT tokens and protects endpoints. Dev 3 learns backend middleware patterns!

**Acceptance Criteria:**
- [ ] File created: `backend/api/middleware/auth.py`
- [ ] Dependency function validates JWT
- [ ] Extracts user_id from token
- [ ] Returns 401 for invalid/missing token
- [ ] Can be added to routes with `dependencies=[Depends(auth)]`
- [ ] Test file validates middleware
- [ ] PR merged

---

## 🕘 9:00 AM - Team Standup

**This is a TEAM CODING DAY. All three developers work together.**

**Setup:**
- One shared screen (or sit together)
- Rotate driver every 30 minutes
- Everyone participates in decisions

**Driver Rotation:**
- 9:30-10:00: Dev 1 (Shabas)
- 10:00-10:30: Dev 2 (Sinan)
- 10:30-11:00: Dev 3 (Marva)
- 11:00-11:30: Back to Dev 1
- Continue rotating...

---

## 🕤 9:30 AM - Start Coding (All Together)

---

## 👨‍💻 Dev 2 (Sinan): AG-28 - User Database Model & Connection

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-28-user-database-model
```

**Jira:** Move AG-28 to "In Progress"

### Step 2: Install Database Dependencies

```bash
cd backend
pip install sqlalchemy psycopg2-binary python-dotenv
pip freeze > requirements.txt
```

### Step 3: Create Database Connection

Create file: `backend/database/connection.py`

```python
"""
Database Connection

Connects to Supabase PostgreSQL using SQLAlchemy.
"""
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
from dotenv import load_dotenv

load_dotenv()

# Get Supabase database URL from environment
DATABASE_URL = os.getenv("DATABASE_URL")

if not DATABASE_URL:
    raise ValueError("DATABASE_URL environment variable not set")

# Create SQLAlchemy engine
engine = create_engine(
    DATABASE_URL,
    pool_pre_ping=True,  # Verify connections before using
    pool_size=5,         # Connection pool size
    max_overflow=10
)

# Session factory
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base class for models
Base = declarative_base()


def get_db():
    """
    Dependency function for FastAPI endpoints.
    
    Yields a database session and ensures cleanup.
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Step 4: Create User Model

Create file: `backend/database/models/__init__.py`

```python
"""Database models."""
from .user import User

__all__ = ["User"]
```

Create file: `backend/database/models/user.py`

```python
"""
User Model

SQLAlchemy model for user accounts.
"""
from datetime import datetime
from sqlalchemy import Column, String, DateTime
from sqlalchemy.dialects.postgresql import UUID
import uuid

from ..connection import Base


class User(Base):
    """User account model."""
    
    __tablename__ = "users"
    
    # Primary key
    id = Column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4,
        nullable=False
    )
    
    # Authentication fields
    email = Column(
        String(255),
        unique=True,
        nullable=False,
        index=True  # Speed up email lookups
    )
    
    password_hash = Column(
        String(255),
        nullable=False
    )
    
    # Metadata
    created_at = Column(
        DateTime(timezone=True),
        default=datetime.utcnow,
        nullable=False
    )
    
    def __repr__(self):
        return f"<User(id={self.id}, email={self.email})>"
    
    def to_dict(self):
        """Convert to dictionary (safe for API responses)."""
        return {
            "id": str(self.id),
            "email": self.email,
            "created_at": self.created_at.isoformat()
        }
```

### Step 5: Create Database Initialization Script

Create file: `backend/database/init_db.py`

```python
"""
Initialize Database

Creates all tables if they don't exist.
Run this once to set up the schema.
"""
from connection import engine, Base
from models import User

def init_db():
    """Create all database tables."""
    print("Creating database tables...")
    Base.metadata.create_all(bind=engine)
    print("✓ Database initialized successfully!")

if __name__ == "__main__":
    init_db()
```

### Step 6: Add DATABASE_URL to .env

Update `backend/.env`:

```bash
# OpenAI
OPENAI_API_KEY=sk-your-key-here

# Supabase Database
DATABASE_URL=postgresql://postgres:[YOUR-PASSWORD]@db.[YOUR-PROJECT-ID].supabase.co:5432/postgres

# JWT Secret (generate with: python -c "import secrets; print(secrets.token_hex(32))")
JWT_SECRET=your-secret-key-here
JWT_ALGORITHM=HS256
JWT_EXPIRY_HOURS=24
```

**NOTE:** Get the DATABASE_URL from Supabase:
1. Go to Project Settings → Database
2. Copy "Connection string" → "URI"
3. Replace `[YOUR-PASSWORD]` with your database password

### Step 7: Initialize Database

```bash
cd backend/database
python init_db.py
```

Expected output:
```
Creating database tables...
✓ Database initialized successfully!
```

### Step 8: Test Connection

Create file: `backend/database/test_connection.py`

```python
"""Test database connection."""
from connection import engine, SessionLocal
from models import User

def test_connection():
    """Test database connectivity."""
    # Test engine connection
    with engine.connect() as conn:
        print("✓ Database connection successful!")
    
    # Test session
    db = SessionLocal()
    user_count = db.query(User).count()
    db.close()
    
    print(f"✓ Users table accessible (count: {user_count})")
    print("✓ All tests passed!")

if __name__ == "__main__":
    test_connection()
```

```bash
python test_connection.py
```

### Step 9: Commit and Push

```bash
cd ../..
git add backend/database/ backend/requirements.txt backend/.env.example
git commit -m "AG-28: Add User database model and connection

- SQLAlchemy connection to Supabase PostgreSQL
- User model with email, password_hash, timestamps
- Database initialization script
- Connection tested successfully"

git push origin feature/AG-28-user-database-model
```

**Jira:** Move AG-28 to "Done"

---

## 👨‍💻 Dev 1 (Shabas): AG-29 - Registration Endpoint

### Step 1: Create Branch

```bash
git checkout shabas-Dev && git pull origin main
git checkout -b feature/AG-29-registration-endpoint
```

**Jira:** Move AG-29 to "In Progress"

### Step 2: Install FastAPI and Auth Dependencies

```bash
cd backend
pip install fastapi uvicorn[standard] pydantic bcrypt pyjwt python-multipart
pip freeze > requirements.txt
```

### Step 3: Create Auth Utilities

Create file: `backend/auth/password.py`

```python
"""
Password Hashing Utilities

Uses bcrypt for secure password hashing.
"""
import bcrypt


def hash_password(password: str) -> str:
    """
    Hash a password using bcrypt.
    
    Args:
        password: Plain text password
        
    Returns:
        Hashed password string
    """
    # Generate salt and hash
    salt = bcrypt.gensalt()
    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
    
    return hashed.decode('utf-8')


def verify_password(password: str, password_hash: str) -> bool:
    """
    Verify a password against its hash.
    
    Args:
        password: Plain text password to verify
        password_hash: Stored hash to check against
        
    Returns:
        True if password matches, False otherwise
    """
    return bcrypt.checkpw(
        password.encode('utf-8'),
        password_hash.encode('utf-8')
    )
```

### Step 4: Create Pydantic Schemas

Create file: `backend/auth/schemas.py`

```python
"""
Auth Request/Response Schemas

Pydantic models for API validation.
"""
from pydantic import BaseModel, EmailStr, Field


class RegisterRequest(BaseModel):
    """Registration request body."""
    email: EmailStr
    password: str = Field(..., min_length=8, max_length=100)


class RegisterResponse(BaseModel):
    """Registration success response."""
    user_id: str
    email: str
    message: str


class LoginRequest(BaseModel):
    """Login request body."""
    email: EmailStr
    password: str


class LoginResponse(BaseModel):
    """Login success response."""
    access_token: str
    token_type: str = "bearer"
    user: dict


class ErrorResponse(BaseModel):
    """Error response."""
    error: str
    detail: str = None
```

### Step 5: Create Registration Endpoint

Create file: `backend/api/routes/auth.py`

```python
"""
Auth Routes

User registration and login endpoints.
"""
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from sqlalchemy.exc import IntegrityError

from database.connection import get_db
from database.models import User
from auth.schemas import (
    RegisterRequest,
    RegisterResponse,
    ErrorResponse
)
from auth.password import hash_password

router = APIRouter(prefix="/api/auth", tags=["auth"])


@router.post(
    "/register",
    response_model=RegisterResponse,
    status_code=status.HTTP_201_CREATED,
    responses={
        400: {"model": ErrorResponse, "description": "Email already registered"},
        422: {"model": ErrorResponse, "description": "Validation error"}
    }
)
def register(
    request: RegisterRequest,
    db: Session = Depends(get_db)
):
    """
    Register a new user account.
    
    Creates a new user with hashed password.
    Email must be unique.
    """
    # Check if email already exists
    existing_user = db.query(User).filter(User.email == request.email).first()
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # Hash password
    password_hash = hash_password(request.password)
    
    # Create user
    new_user = User(
        email=request.email,
        password_hash=password_hash
    )
    
    try:
        db.add(new_user)
        db.commit()
        db.refresh(new_user)
    except IntegrityError:
        db.rollback()
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    return RegisterResponse(
        user_id=str(new_user.id),
        email=new_user.email,
        message="User registered successfully"
    )
```

### Step 6: Create FastAPI App

Create file: `backend/main.py`

```python
"""
Resume Agent FastAPI Application

Main entry point for the backend API.
"""
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from api.routes.auth import router as auth_router

# Create FastAPI app
app = FastAPI(
    title="Resume Agent API",
    description="AI-powered resume optimization API",
    version="1.0.0"
)

# CORS middleware (allow frontend to call API)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Vite dev server
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Register routes
app.include_router(auth_router)


@app.get("/")
def root():
    """Health check endpoint."""
    return {
        "status": "ok",
        "service": "Resume Agent API",
        "version": "1.0.0"
    }


@app.get("/health")
def health():
    """Detailed health check."""
    return {
        "status": "healthy",
        "database": "connected",  # TODO: Add actual DB check
        "api": "operational"
    }
```

### Step 7: Test Registration

Start the server:

```bash
cd backend
uvicorn main:app --reload --port 8000
```

Test with curl:

```bash
# Register a new user
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password123"}'
```

Expected response:
```json
{
  "user_id": "uuid-here",
  "email": "test@example.com",
  "message": "User registered successfully"
}
```

Test duplicate email (should fail):
```bash
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "different123"}'
```

Expected response:
```json
{
  "detail": "Email already registered"
}
```

### Step 8: Commit and Push

```bash
git add backend/
git commit -m "AG-29: Add user registration endpoint

- FastAPI app setup with CORS
- Registration endpoint with validation
- Password hashing with bcrypt
- Pydantic schemas for request/response
- Duplicate email handling"

git push origin feature/AG-29-registration-endpoint
```

**Jira:** Move AG-29 to "Done"

---

## 👨‍💻 Dev 2 (Sinan): AG-30 - Login & JWT Endpoint

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-30-login-jwt-endpoint
```

**Jira:** Move AG-30 to "In Progress"

### Step 2: Create JWT Utilities

Create file: `backend/auth/jwt.py`

```python
"""
JWT Token Utilities

Create and verify JWT access tokens.
"""
import os
from datetime import datetime, timedelta
from typing import Dict, Optional
import jwt
from dotenv import load_dotenv

load_dotenv()

JWT_SECRET = os.getenv("JWT_SECRET")
JWT_ALGORITHM = os.getenv("JWT_ALGORITHM", "HS256")
JWT_EXPIRY_HOURS = int(os.getenv("JWT_EXPIRY_HOURS", "24"))

if not JWT_SECRET:
    raise ValueError("JWT_SECRET environment variable not set")


def create_access_token(user_id: str, email: str) -> str:
    """
    Create a JWT access token.
    
    Args:
        user_id: User's UUID as string
        email: User's email
        
    Returns:
        Encoded JWT token string
    """
    # Set expiration time
    expires_at = datetime.utcnow() + timedelta(hours=JWT_EXPIRY_HOURS)
    
    # Create payload
    payload = {
        "sub": user_id,      # Subject (user ID)
        "email": email,
        "exp": expires_at,   # Expiration time
        "iat": datetime.utcnow()  # Issued at
    }
    
    # Encode token
    token = jwt.encode(payload, JWT_SECRET, algorithm=JWT_ALGORITHM)
    
    return token


def decode_access_token(token: str) -> Optional[Dict]:
    """
    Decode and verify a JWT token.
    
    Args:
        token: JWT token string
        
    Returns:
        Decoded payload if valid, None if invalid/expired
    """
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        return None  # Token expired
    except jwt.InvalidTokenError:
        return None  # Invalid token
```

### Step 3: Add Login Endpoint

Update file: `backend/api/routes/auth.py` (add to existing file):

```python
from auth.jwt import create_access_token
from auth.password import verify_password

# ... existing register endpoint ...

@router.post(
    "/login",
    response_model=LoginResponse,
    responses={
        401: {"model": ErrorResponse, "description": "Invalid credentials"}
    }
)
def login(
    request: LoginRequest,
    db: Session = Depends(get_db)
):
    """
    Login and receive JWT access token.
    
    Verifies credentials and returns JWT for authenticated requests.
    """
    # Find user by email
    user = db.query(User).filter(User.email == request.email).first()
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Verify password
    if not verify_password(request.password, user.password_hash):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Create JWT token
    access_token = create_access_token(
        user_id=str(user.id),
        email=user.email
    )
    
    return LoginResponse(
        access_token=access_token,
        token_type="bearer",
        user=user.to_dict()
    )
```

### Step 4: Test Login

Restart the server:

```bash
cd backend
uvicorn main:app --reload --port 8000
```

Test login:

```bash
# Login with correct credentials
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password123"}'
```

Expected response:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "user": {
    "id": "uuid-here",
    "email": "test@example.com",
    "created_at": "2024-01-15T10:30:00"
  }
}
```

Test wrong password (should fail):
```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "wrongpassword"}'
```

Expected response:
```json
{
  "detail": "Invalid email or password"
}
```

### Step 5: Commit and Push

```bash
git add backend/auth/jwt.py backend/api/routes/auth.py
git commit -m "AG-30: Add login endpoint with JWT tokens

- JWT creation and verification utilities
- Login endpoint with credential validation
- Returns access token + user info
- Invalid credentials handling"

git push origin feature/AG-30-login-jwt-endpoint
```

**Jira:** Move AG-30 to "Done"

---

## 👩‍💻 Dev 3 (Marva): AG-31 - Auth Middleware

### Step 1: Create Branch

```bash
git checkout marva-Dev && git pull origin main
git checkout -b feature/AG-31-auth-middleware
```

**Jira:** Move AG-31 to "In Progress"

### Step 2: Create Auth Dependency

Create file: `backend/auth/dependencies.py`

```python
"""
Auth Dependencies

FastAPI dependencies for protecting routes.
"""
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.orm import Session

from database.connection import get_db
from database.models import User
from .jwt import decode_access_token

# Bearer token scheme
security = HTTPBearer()


def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
) -> User:
    """
    Verify JWT token and return current user.
    
    Use this as a dependency to protect routes:
    ```
    @app.get("/protected")
    def protected_route(user: User = Depends(get_current_user)):
        return {"user_id": user.id}
    ```
    
    Raises:
        HTTPException: If token is invalid or user not found
    """
    token = credentials.credentials
    
    # Decode token
    payload = decode_access_token(token)
    
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    # Get user from database
    user_id = payload.get("sub")
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    return user
```

### Step 3: Create Protected Test Endpoint

Create file: `backend/api/routes/user.py`

```python
"""
User Routes

Protected endpoints requiring authentication.
"""
from fastapi import APIRouter, Depends

from database.models import User
from auth.dependencies import get_current_user

router = APIRouter(prefix="/api/user", tags=["user"])


@router.get("/me")
def get_current_user_info(current_user: User = Depends(get_current_user)):
    """
    Get current user's information.
    
    Protected endpoint - requires valid JWT token.
    """
    return {
        "user": current_user.to_dict(),
        "message": "Authentication successful"
    }


@router.get("/profile")
def get_profile(current_user: User = Depends(get_current_user)):
    """
    Get user profile (placeholder for Sprint 2).
    
    Protected endpoint demonstrating auth middleware.
    """
    return {
        "user_id": str(current_user.id),
        "email": current_user.email,
        "member_since": current_user.created_at.isoformat(),
        "resume_count": 0,  # TODO: Count user's optimization runs
        "status": "active"
    }
```

### Step 4: Register User Routes

Update `backend/main.py`:

```python
from api.routes.auth import router as auth_router
from api.routes.user import router as user_router  # Add this

# ... existing code ...

# Register routes
app.include_router(auth_router)
app.include_router(user_router)  # Add this
```

### Step 5: Test Auth Middleware

Restart server:

```bash
cd backend
uvicorn main:app --reload --port 8000
```

Test without token (should fail):

```bash
curl http://localhost:8000/api/user/me
```

Expected response:
```json
{
  "detail": "Not authenticated"
}
```

Test with token (should succeed):

```bash
# First, login to get token
TOKEN=$(curl -s -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password123"}' \
  | jq -r '.access_token')

# Use token to access protected endpoint
curl http://localhost:8000/api/user/me \
  -H "Authorization: Bearer $TOKEN"
```

Expected response:
```json
{
  "user": {
    "id": "uuid-here",
    "email": "test@example.com",
    "created_at": "2024-01-15T10:30:00"
  },
  "message": "Authentication successful"
}
```

Test profile endpoint:

```bash
curl http://localhost:8000/api/user/profile \
  -H "Authorization: Bearer $TOKEN"
```

Expected response:
```json
{
  "user_id": "uuid-here",
  "email": "test@example.com",
  "member_since": "2024-01-15T10:30:00",
  "resume_count": 0,
  "status": "active"
}
```

### Step 6: Commit and Push

```bash
git add backend/auth/dependencies.py backend/api/routes/user.py backend/main.py
git commit -m "AG-31: Add auth middleware and protected routes

- get_current_user dependency for route protection
- JWT validation and user lookup
- Protected /api/user/me and /api/user/profile endpoints
- Comprehensive auth flow testing"

git push origin feature/AG-31-auth-middleware
```

**Jira:** Move AG-31 to "Done"

---

## ✅ Day 6: Manual Verification (CRITICAL!)

### Terminal Tests

```bash
# 1. Test Registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "verify@test.com", "password": "testpass123"}'

# 2. Test Login
TOKEN=$(curl -s -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "verify@test.com", "password": "testpass123"}' \
  | jq -r '.access_token')

echo "Token: $TOKEN"

# 3. Test Protected Endpoint
curl http://localhost:8000/api/user/me \
  -H "Authorization: Bearer $TOKEN"

# 4. Test Invalid Token
curl http://localhost:8000/api/user/me \
  -H "Authorization: Bearer invalid-token-here"
```

### What Must Work

| Check | Expected |
|-------|----------|
| Register new user | ✓ Returns user_id |
| Duplicate email fails | ✓ Returns 400 error |
| Login with correct password | ✓ Returns JWT token |
| Login with wrong password | ✓ Returns 401 error |
| Access protected route with token | ✓ Returns user data |
| Access protected route without token | ✓ Returns 401 error |
| Database connection | ✓ No connection errors |

### If Something Fails

Common issues:
1. **Database connection error** → Check DATABASE_URL in .env
2. **JWT secret error** → Check JWT_SECRET in .env
3. **Import errors** → Check all `__init__.py` files
4. **CORS errors in browser** → Check CORS middleware config
5. **bcrypt install error** → Use `pip install bcrypt` or `bcrypt-cffi`

---

## 📁 Folder Structure After Day 6

```
backend/
├── main.py                  ✓ FastAPI app
├── requirements.txt         ✓ Updated dependencies
├── .env                     ✓ DATABASE_URL, JWT_SECRET added
│
├── database/
│   ├── connection.py        ✓ NEW - SQLAlchemy setup
│   ├── init_db.py          ✓ NEW - Table creation
│   ├── test_connection.py  ✓ NEW - Connection test
│   └── models/
│       ├── __init__.py     ✓ NEW
│       └── user.py         ✓ NEW - User model
│
├── auth/
│   ├── password.py         ✓ NEW - bcrypt hashing
│   ├── jwt.py              ✓ NEW - JWT creation/verification
│   ├── schemas.py          ✓ NEW - Pydantic models
│   └── dependencies.py     ✓ NEW - Auth middleware
│
└── api/
    └── routes/
        ├── auth.py         ✓ NEW - Register/login endpoints
        └── user.py         ✓ NEW - Protected endpoints
```

---

## 🔄 Evening Standup

**What did we accomplish today?**

| Dev | Task | Status |
|-----|------|--------|
| Sinan | AG-28: User Database Model | ✅ Complete |
| Shabas | AG-29: Registration Endpoint | ✅ Complete |
| Sinan | AG-30: Login & JWT | ✅ Complete |
| Marva | AG-31: Auth Middleware | ✅ Complete |

**Key achievements:**
- ✅ Database connected to Supabase PostgreSQL
- ✅ User registration with password hashing
- ✅ Login with JWT token generation
- ✅ Auth middleware protecting routes
- ✅ End-to-end auth flow tested!

**Blockers:** None

**Tomorrow (Day 7):**
- AG-32: POST /optimize endpoint
- AG-33: GET /runs/{id} endpoint
- AG-34: GET /runs endpoint
- AG-35: Create agent run in database

**Notes:**
- Auth system is secure and working
- Ready to connect agent workflow to API
- Cross-training went well (Shabas on backend, Marva on auth)

---

    Convenience function to run the full optimization.
    
    Args:
        job_description: The job posting text
        resume: The user's resume text
        user_id: Optional user identifier
        run_id: Optional run identifier (generated if not provided)
        
    Returns:
        Dict with optimization results
    """
    import uuid
    
    if run_id is None:
        run_id = str(uuid.uuid4())
    
    # Create initial state
    initial_state = {
        "run_id": run_id,
        "user_id": user_id,
        "job_description": job_description,
        "original_resume": resume,
        "job_requirements": None,
        "resume_analysis": None,
        "ats_score_before": None,
        "ats_score_after": None,
        "improvement_delta": None,
        "score_history": [],
        "improvement_plan": None,
        "modified_resume": None,
        "applied_changes": [],
        "decision_log": [],
        "cover_letter": None,
        "iteration_count": 0,
        "max_iterations": 3,
        "final_status": "pending"
    }
    
    # Run workflow
    result = agent_app.invoke(initial_state)
    
    # Update final status
    result["final_status"] = "completed"
    
    return result
```

**⏰ 10:00 - SWITCH DRIVER to Dev 2**

### Step 3: Create Comprehensive Test

Create file: `backend/agent/test_workflow.py`

```python
"""
End-to-End Workflow Tests

Run with: python test_workflow.py

This tests the complete agent workflow from start to finish.
"""
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Verify API key exists
if not os.getenv("OPENAI_API_KEY"):
    print("❌ Error: OPENAI_API_KEY not set in .env file")
    print("Add: OPENAI_API_KEY=sk-your-key-here")
    exit(1)

from workflow import run_optimization, agent_app


# Sample job description
SAMPLE_JOB_DESCRIPTION = """
Senior Backend Engineer - FinTech Startup

We're looking for a talented Senior Backend Engineer to join our growing team.

Requirements:
- 5+ years of experience in backend development
- Strong proficiency in Python
- Experience with Django or FastAPI
- PostgreSQL database experience
- Docker and containerization knowledge
- AWS cloud experience preferred
- Experience building REST APIs
- Understanding of microservices architecture

Nice to Have:
- Kubernetes experience
- GraphQL knowledge
- Message queues (RabbitMQ, Kafka)

Responsibilities:
- Design and implement scalable backend services
- Build and maintain REST APIs
- Write comprehensive tests
- Participate in code reviews
- Mentor junior developers
- Contribute to architecture decisions

We value innovation, collaboration, and continuous learning.
"""

# Sample resume (intentionally not perfect to show improvement)
SAMPLE_RESUME = """
John Smith
Software Developer
john.smith@email.com | (555) 123-4567

SUMMARY
Experienced software developer with 4 years of experience building web applications.

EXPERIENCE

Software Developer | TechCorp Inc | 2020 - Present
- Developed web applications using Python
- Worked with databases
- Fixed bugs and added features
- Participated in team meetings

Junior Developer | StartupXYZ | 2019 - 2020
- Wrote code for backend systems
- Learned about APIs
- Helped senior developers

SKILLS
Python, JavaScript, SQL, Git

EDUCATION
Bachelor of Science in Computer Science
State University, 2019
"""


def test_full_workflow():
    """Test the complete optimization workflow."""
    print("=" * 60)
    print("RESUME OPTIMIZATION AGENT - FULL WORKFLOW TEST")
    print("=" * 60)
    
    print("\n📝 Running optimization...")
    print("This will make several API calls to OpenAI.\n")
    
    result = run_optimization(
        job_description=SAMPLE_JOB_DESCRIPTION,
        resume=SAMPLE_RESUME,
        user_id="test-user",
        run_id="test-run-001"
    )
    
    # === VERIFY RESULTS ===
    
    print("\n" + "=" * 60)
    print("RESULTS")
    print("=" * 60)
    
    # 1. Check job requirements extracted
    print("\n📋 Job Requirements Extracted:")
    reqs = result.get("job_requirements", {})
    print(f"   Must-have skills: {reqs.get('must_have_skills', [])[:5]}")
    print(f"   Keywords: {reqs.get('keywords', [])[:5]}")
    assert reqs, "Job requirements should be extracted"
    
    # 2. Check resume analyzed
    print("\n📊 Resume Analysis:")
    analysis = result.get("resume_analysis", {})
    print(f"   Current skills: {analysis.get('current_skills', [])[:5]}")
    print(f"   Experience level: {analysis.get('experience_level', 'N/A')}")
    assert analysis, "Resume should be analyzed"
    
    # 3. Check scores
    print("\n📈 Scores:")
    before = result.get("ats_score_before", 0)
    after = result.get("ats_score_after", 0)
    delta = result.get("improvement_delta", 0)
    print(f"   Before: {before}")
    print(f"   After:  {after}")
    print(f"   Delta:  {delta:+.1f}")
    assert before is not None, "Should have initial score"
    assert after is not None, "Should have final score"
    
    # 4. Check improvement plan
    print("\n📝 Improvement Plan:")
    plan = result.get("improvement_plan", {})
    changes = plan.get("priority_changes", [])
    print(f"   Changes planned: {len(changes)}")
    for i, change in enumerate(changes[:3], 1):
        print(f"   {i}. {change}")
    
    # 5. Check modified resume exists
    print("\n📄 Modified Resume:")
    modified = result.get("modified_resume", "")
    print(f"   Length: {len(modified)} characters")
    print(f"   First 200 chars: {modified[:200]}...")
    assert modified, "Should have modified resume"
    
    # 6. Check decision log
    print("\n🔍 Decision Log:")
    decisions = result.get("decision_log", [])
    print(f"   Total decisions: {len(decisions)}")
    for d in decisions:
        print(f"   - {d.get('node', '?')}: {d.get('action', '?')}")
    
    # 7. Verify improvement (main goal!)
    print("\n" + "=" * 60)
    print("FINAL VERDICT")
    print("=" * 60)
    
    if delta > 0:
        print(f"\n✅ SUCCESS! Score improved by {delta:+.1f} points")
        print(f"   From {before} → {after}")
    elif delta == 0:
        print(f"\n⚠️ Warning: Score didn't change ({before} → {after})")
    else:
        print(f"\n❌ Score decreased by {delta:.1f} points")
    
    # Assertions
    assert result["final_status"] == "completed"
    assert len(decisions) >= 5, "Should have at least 5 decision log entries"
    
    print("\n🎉 WORKFLOW TEST COMPLETED!")
    return result


def test_workflow_state_integrity():
    """Test that state flows correctly through nodes."""
    print("\n" + "=" * 60)
    print("STATE INTEGRITY TEST")
    print("=" * 60)
    
    # Create minimal state
    initial_state = {
        "run_id": "integrity-test",
        "user_id": "test",
        "job_description": "Python developer needed with Django experience",
        "original_resume": "John Doe, Python developer with 3 years experience",
        "job_requirements": None,
        "resume_analysis": None,
        "ats_score_before": None,
        "ats_score_after": None,
        "improvement_delta": None,
        "score_history": [],
        "improvement_plan": None,
        "modified_resume": None,
        "applied_changes": [],
        "decision_log": [],
        "cover_letter": None,
        "iteration_count": 0,
        "max_iterations": 3,
        "final_status": "pending"
    }
    
    result = agent_app.invoke(initial_state)
    
    # Check all expected fields are populated
    assert result["job_requirements"] is not None, "job_requirements should be set"
    assert result["resume_analysis"] is not None, "resume_analysis should be set"
    assert result["ats_score_before"] is not None, "ats_score_before should be set"
    assert result["ats_score_after"] is not None, "ats_score_after should be set"
    assert result["improvement_plan"] is not None, "improvement_plan should be set"
    assert result["modified_resume"] is not None, "modified_resume should be set"
    assert len(result["score_history"]) >= 2, "Should have at least 2 scores in history"
    assert result["iteration_count"] >= 1, "Should have at least 1 iteration"
    
    print("✓ All state fields populated correctly!")
    print("✓ State integrity test passed!")


if __name__ == "__main__":
    # Run full workflow test
    test_full_workflow()
    print()
    
    # Run state integrity test
    test_workflow_state_integrity()
    
    print("\n" + "=" * 60)
    print("🎉 ALL WORKFLOW TESTS PASSED!")
    print("=" * 60)
```

**⏰ 10:30 - SWITCH DRIVER to Dev 3**

### Step 4: Run the Tests

```bash
cd backend

# Make sure .env has OPENAI_API_KEY
cat .env | grep OPENAI

# Run the workflow test
cd agent
python test_workflow.py
```

**Expected Output:**
```
============================================================
RESUME OPTIMIZATION AGENT - FULL WORKFLOW TEST
============================================================

📝 Running optimization...
This will make several API calls to OpenAI.

============================================================
RESULTS
============================================================

📋 Job Requirements Extracted:
   Must-have skills: ['Python', 'Django', 'FastAPI', 'PostgreSQL', 'Docker']
   Keywords: ['backend', 'REST APIs', 'microservices', 'scalable']

📊 Resume Analysis:
   Current skills: ['Python', 'JavaScript', 'SQL', 'Git']
   Experience level: mid

📈 Scores:
   Before: 42.5
   After:  68.3
   Delta:  +25.8

📝 Improvement Plan:
   Changes planned: 5
   1. Add more specific technical skills
   2. Quantify achievements with metrics
   3. Include relevant keywords

📄 Modified Resume:
   Length: 1245 characters
   First 200 chars: John Smith
Senior Backend Engineer...

🔍 Decision Log:
   Total decisions: 6
   - extract_job_requirements: extracted_requirements
   - analyze_resume: analyzed_resume_content
   - score_initial: calculated_initial_score
   - plan_improvements: created_improvement_plan
   - modify_resume: applied_improvements
   - score_modified: calculated_modified_score

============================================================
FINAL VERDICT
============================================================

✅ SUCCESS! Score improved by +25.8 points
   From 42.5 → 68.3

🎉 WORKFLOW TEST COMPLETED!

============================================================
STATE INTEGRITY TEST
============================================================
✓ All state fields populated correctly!
✓ State integrity test passed!

============================================================
🎉 ALL WORKFLOW TESTS PASSED!
============================================================
```

**⏰ 11:00 - SWITCH DRIVER back to Dev 1**

### Step 5: Update Nodes __init__.py

Update file: `backend/agent/nodes/__init__.py`

```python
"""
LangGraph Nodes for Resume Optimization Agent

Each node is a function that takes state and returns updates.
"""
from .job_requirements import extract_job_requirements
from .resume_analysis import analyze_resume
from .scoring import score_initial, score_modified
from .planning import plan_improvements
from .modification import modify_resume

__all__ = [
    "extract_job_requirements",
    "analyze_resume",
    "score_initial",
    "score_modified",
    "plan_improvements",
    "modify_resume"
]
```

### Step 6: Commit Everything

```bash
cd ../..  # Back to resume-agent root

## 📊 Sprint 1 Progress

| Day | Focus | Tasks | Status |
|-----|-------|-------|--------|
| 1 | Setup | AG-7 to AG-13 | ✅ Complete |
| 2 | First Node | AG-14 to AG-17 | ✅ Complete |
| 3 | Three Nodes | AG-18 to AG-20 | ✅ Complete |
| 4 | Two Nodes | AG-21 to AG-23 | ✅ Complete |
| 5 | Workflow | AG-24 to AG-26 | ✅ Complete |
| **6** | **Auth Backend** | **AG-28 to AG-31** | **✅ Complete** |
| 7 | Agent Endpoints | AG-32 to AG-35 | 📅 Tomorrow |

**Sprint 1 Progress:** 30/45 tasks (67%) ✅

---

**← [Day 5](./day-05.md)** | **[Day 7: Agent API Endpoints →](./day-07.md)**
