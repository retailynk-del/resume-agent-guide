# Day 11: Sprint 2 - Supabase Integration

> **Date:** Sprint 2, Day 1  
> **Focus:** Migrate from in-memory to Supabase database  
> **Vertical Slice:** Slice 5 - Persistent data

---

## 🎯 Today's Goal

By end of today:
- ✅ User model in database
- ✅ Registration saves to Supabase
- ✅ Login reads from Supabase
- ✅ All existing tests pass

---

## 📋 Morning: Sprint 2 Kickoff

### Sprint 2 Goals
1. Database persistence
2. Iterative agent (conditional edges)
3. Cover letter generation
4. Run history
5. Polish and improvements

---

## 📋 Jira Tasks

### Task AG-28: User Database Model

| Field | Value |
|-------|-------|
| **Task ID** | AG-28 |
| **Title** | User Database Model |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |

**Description:**
Create SQLAlchemy model for users and migrate auth to use database.

**Acceptance Criteria:**
- [ ] User model with id, email, password_hash, created_at
- [ ] Registration saves to database
- [ ] Login reads from database
- [ ] Existing users can still login

---

### Task AG-29: Run History Model

| Field | Value |
|-------|-------|
| **Task ID** | AG-29 |
| **Title** | Agent Run History Model |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |

**Description:**
Create model to store optimization runs for history feature.

**Acceptance Criteria:**
- [ ] Run model with id, user_id, scores, status
- [ ] Runs saved after completion
- [ ] Can query runs by user

---

## 👨‍💻 Dev 2 (Sinan): AG-28 - User Database Model

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-28-user-model
```

**Jira:** Move AG-28 to "In Progress"

### Step 2: Create Models Directory

```bash
mkdir -p backend/database/models
touch backend/database/models/__init__.py
```

### Step 3: Create User Model

Create file: `backend/database/models/user.py`

```python
"""
User Database Model

Stores user authentication data in Supabase PostgreSQL.
"""
from datetime import datetime
import uuid

from sqlalchemy import Column, String, DateTime, Text
from sqlalchemy.dialects.postgresql import UUID

from database.connection import Base


class User(Base):
    """User model for authentication."""
    
    __tablename__ = "users"
    
    id = Column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4
    )
    
    email = Column(
        String(255),
        unique=True,
        nullable=False,
        index=True
    )
    
    password_hash = Column(
        Text,
        nullable=False
    )
    
    created_at = Column(
        DateTime,
        default=datetime.utcnow,
        nullable=False
    )
    
    def __repr__(self):
        return f"<User {self.email}>"
    
    def to_dict(self):
        """Convert to dictionary for JSON serialization."""
        return {
            "id": str(self.id),
            "email": self.email,
            "created_at": self.created_at.isoformat()
        }
```

### Step 4: Update Models Init

Update file: `backend/database/models/__init__.py`

```python
"""Database models."""
from .user import User

__all__ = ["User"]
```

### Step 5: Create Database Migration Script

Create file: `backend/database/init_db.py`

```python
"""
Database Initialization Script

Run this once to create tables in Supabase.
"""
from connection import engine, Base
from models.user import User

def init_database():
    """Create all tables."""
    if engine is None:
        print("❌ DATABASE_URL not configured")
        return False
    
    try:
        Base.metadata.create_all(bind=engine)
        print("✓ Database tables created successfully")
        return True
    except Exception as e:
        print(f"❌ Error creating tables: {e}")
        return False


if __name__ == "__main__":
    from dotenv import load_dotenv
    load_dotenv()
    init_database()
```

### Step 6: Update Auth Router

Update file: `backend/api/auth.py`

```python
"""
Authentication API Endpoints

Uses Supabase PostgreSQL for user storage.
"""
import os
from datetime import datetime, timedelta
from typing import Optional
import uuid

from fastapi import APIRouter, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
from jose import jwt, JWTError
from sqlalchemy.orm import Session

from database.connection import get_db, SessionLocal
from database.models.user import User

router = APIRouter(prefix="/api/auth", tags=["Authentication"])

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key-change-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_HOURS = 24


# Pydantic models (same as before)
class UserRegister(BaseModel):
    email: EmailStr
    password: str

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    user_id: str

class UserResponse(BaseModel):
    id: str
    email: str

class MessageResponse(BaseModel):
    message: str
    user_id: Optional[str] = None


# Helper functions
def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(hours=ACCESS_TOKEN_EXPIRE_HOURS)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None


# API Endpoints
@router.post("/register", response_model=MessageResponse, status_code=201)
def register(user: UserRegister, db: Session = Depends(get_db)):
    """Register a new user."""
    # Check if email exists
    existing = db.query(User).filter(User.email == user.email.lower()).first()
    if existing:
        raise HTTPException(
            status_code=400,
            detail="Email already registered"
        )
    
    if len(user.password) < 6:
        raise HTTPException(
            status_code=400,
            detail="Password must be at least 6 characters"
        )
    
    # Create user
    new_user = User(
        email=user.email.lower(),
        password_hash=hash_password(user.password)
    )
    
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    
    return MessageResponse(
        message="User registered successfully",
        user_id=str(new_user.id)
    )


@router.post("/login", response_model=TokenResponse)
def login(user: UserLogin, db: Session = Depends(get_db)):
    """Login and receive JWT token."""
    db_user = db.query(User).filter(User.email == user.email.lower()).first()
    
    if not db_user:
        raise HTTPException(
            status_code=401,
            detail="Invalid email or password"
        )
    
    if not verify_password(user.password, db_user.password_hash):
        raise HTTPException(
            status_code=401,
            detail="Invalid email or password"
        )
    
    access_token = create_access_token(
        data={"sub": str(db_user.id), "email": db_user.email}
    )
    
    return TokenResponse(
        access_token=access_token,
        token_type="bearer",
        expires_in=ACCESS_TOKEN_EXPIRE_HOURS * 3600,
        user_id=str(db_user.id)
    )


@router.get("/me", response_model=UserResponse)
def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
):
    """Get current user info."""
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    user_id = payload.get("sub")
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    
    return UserResponse(id=str(user.id), email=user.email)


def get_current_user_id(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> str:
    """Dependency to get current user ID from token."""
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return payload.get("sub")
```

### Step 7: Initialize Database

```bash
cd backend/database
python init_db.py

# Expected: ✓ Database tables created successfully
```

### Step 8: Test

```bash
cd backend
uvicorn main:app --reload

# Test registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "db-test@test.com", "password": "password123"}'

# Test login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "db-test@test.com", "password": "password123"}'
```

### Step 9: Commit and PR

```bash
git add backend/database/ backend/api/auth.py
git commit -m "AG-28: Migrate auth to Supabase

- Create User SQLAlchemy model
- Update auth endpoints to use database
- Add database init script
- All existing auth tests pass"

git push origin feature/AG-28-user-model
```

**Jira:** Move AG-28 to "Done"

---

## 👨‍💻 Dev 1 (Shabas): AG-29 - Run History Model

### Step 1: Create Branch

```bash
git checkout dev-shabas && git pull origin main
git checkout -b feature/AG-29-run-model
```

**Jira:** Move AG-29 to "In Progress"

### Step 2: Create Run Model

Create file: `backend/database/models/run.py`

```python
"""
Agent Run Model

Stores optimization run history.
"""
from datetime import datetime
import uuid

from sqlalchemy import Column, String, DateTime, Float, Text, ForeignKey, JSON
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.orm import relationship

from database.connection import Base


class AgentRun(Base):
    """Stores each optimization run."""
    
    __tablename__ = "agent_runs"
    
    id = Column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4
    )
    
    user_id = Column(
        UUID(as_uuid=True),
        ForeignKey("users.id"),
        nullable=True,  # Allow anonymous runs
        index=True
    )
    
    job_description = Column(Text, nullable=False)
    original_resume = Column(Text, nullable=False)
    modified_resume = Column(Text)
    
    ats_score_before = Column(Float)
    ats_score_after = Column(Float)
    improvement_delta = Column(Float)
    
    job_requirements = Column(JSON)
    improvement_plan = Column(JSON)
    
    status = Column(
        String(20),
        default="pending"  # pending, running, completed, failed
    )
    
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
    
    def to_dict(self):
        return {
            "id": str(self.id),
            "user_id": str(self.user_id) if self.user_id else None,
            "ats_score_before": self.ats_score_before,
            "ats_score_after": self.ats_score_after,
            "improvement_delta": self.improvement_delta,
            "status": self.status,
            "created_at": self.created_at.isoformat() if self.created_at else None
        }
    
    def to_full_dict(self):
        """Include full content for detail view."""
        data = self.to_dict()
        data.update({
            "job_description": self.job_description,
            "original_resume": self.original_resume,
            "modified_resume": self.modified_resume,
            "job_requirements": self.job_requirements,
            "improvement_plan": self.improvement_plan
        })
        return data
```

### Step 3: Update Models Init

Update file: `backend/database/models/__init__.py`

```python
"""Database models."""
from .user import User
from .run import AgentRun

__all__ = ["User", "AgentRun"]
```

### Step 4: Update Init Script

Update file: `backend/database/init_db.py`

```python
"""Database initialization."""
from connection import engine, Base
from models.user import User
from models.run import AgentRun

def init_database():
    """Create all tables."""
    if engine is None:
        print("❌ DATABASE_URL not configured")
        return False
    
    try:
        Base.metadata.create_all(bind=engine)
        print("✓ All database tables created")
        return True
    except Exception as e:
        print(f"❌ Error: {e}")
        return False

if __name__ == "__main__":
    from dotenv import load_dotenv
    load_dotenv()
    init_database()
```

### Step 5: Run Migration

```bash
cd backend/database
python init_db.py
# Should show: ✓ All database tables created
```

### Step 6: Commit and PR

```bash
git add backend/database/
git commit -m "AG-29: Create AgentRun model

- Store optimization runs in database
- Include all scores and content
- Add status tracking
- Link to user (optional for anonymous)"

git push origin feature/AG-29-run-model
```

**Jira:** Move AG-29 to "Done"

---

## ✅ Day 11: Manual Verification

### Database Verification (Supabase Dashboard)

1. Login to Supabase: https://supabase.com
2. Open your project
3. Go to Table Editor
4. Verify `users` table exists with correct columns
5. Verify `agent_runs` table exists

### Terminal Tests

```bash
# Test user registration still works
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "sprint2-test@test.com", "password": "Test1234"}'

# Check Supabase - user should appear in users table
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-28: User Model | Dev 2 | 5 | ✓ Done |
| AG-29: Run Model | Dev 1 | 5 | ✓ Done |

**Total:** 10 points

---

**← [Day 10](./day-10.md)** | **[Day 12: Save Run History →](./day-12.md)**
