# Day 7: Auth Frontend

> **Date:** Sprint 1, Day 7  
> **Focus:** User authentication UI (login + register pages)  
> **Vertical Slice:** Full auth flow works in UI

---

## 🎯 Today's Goal

By end of today:
- ✅ Login page component ready (Dev 3)
- ✅ Register page component ready (Dev 3)
- ✅ Auth API service module created (Dev 1 - learning frontend!)
- ✅ Protected routes working (Dev 2 - cross-training!)
- ✅ Full flow: Register in UI → Login → Get token → Access dashboard

---

## 📋 Morning: Add These Tasks to Jira

---

### Task AG-32: Login Page Component

| Field | Value |
|-------|-------|
| **Task ID** | AG-32 |
| **Title** | Login Page Component |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a polished login page component with email/password form, validation, and error handling.

**Acceptance Criteria:**
- [ ] File created: `frontend/src/pages/Login.jsx`
- [ ] Email and password input fields
- [ ] Client-side validation
- [ ] Error message display
- [ ] Loading state during submission
- [ ] Styled with modern UI
- [ ] PR merged

---

### Task AG-33: Register Page Component

| Field | Value |
|-------|-------|
| **Task ID** | AG-33 |
| **Title** | Register Page Component |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a registration page component with email, password, and confirm password fields. Includes client-side validation.

**Acceptance Criteria:**
- [ ] File created: `frontend/src/pages/Register.jsx`
- [ ] Email, password, and confirm password fields
- [ ] Password strength validation
- [ ] Passwords must match
- [ ] Success message and redirect
- [ ] Styled consistently with login page
- [ ] PR merged

---

### Task AG-34: Auth API Service Module

| Field | Value |
|-------|-------|
| **Task ID** | AG-34 |
| **Title** | Auth API Service Module |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create frontend API service module for authentication. Handles API calls to backend auth endpoints. Dev 1 learns frontend code!

**Acceptance Criteria:**
- [ ] File created: `frontend/src/services/api.js`
- [ ] Functions: register(), login(), logout()
- [ ] JWT token storage in localStorage
- [ ] Error handling for failed requests
- [ ] Integrates with backend endpoints
- [ ] PR merged

---

### Task AG-35: Protected Routes Setup

| Field | Value |
|-------|-------|
| **Task ID** | AG-35 |
| **Title** | Protected Routes Setup |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create React Router protected route wrapper that redirects to login if user is not authenticated. Dev 2 learns frontend routing!

**Acceptance Criteria:**
- [ ] File created: `frontend/src/components/ProtectedRoute.jsx`  
- [ ] Checks for valid JWT token
- [ ] Redirects to /login if not authenticated
- [ ] Allows access if authenticated
- [ ] Applied to dashboard route
- [ ] PR merged

---

## 👨‍💻 Dev 2 (Sinan): AG-25 - Auth Endpoints

### Step 1: Create Branch

```bash
cd resume-agent
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-25-auth-endpoints
```

**Jira:** Move AG-25 to "In Progress"

### Step 2: Create Auth Router

Create file: `backend/api/auth.py`

```python
"""
Authentication API Endpoints

Handles user registration, login, and JWT token management.
"""
import os
from datetime import datetime, timedelta
from typing import Optional

from fastapi import APIRouter, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
from jose import jwt, JWTError

# Create router
router = APIRouter(prefix="/api/auth", tags=["Authentication"])

# Security setup
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

# Configuration
SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key-change-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_HOURS = 24

# In-memory user storage (will migrate to Supabase)
# Format: {email: {"id": str, "email": str, "password_hash": str}}
users_db = {}


# === Pydantic Models ===

class UserRegister(BaseModel):
    """Request model for user registration."""
    email: EmailStr
    password: str
    
    class Config:
        json_schema_extra = {
            "example": {
                "email": "user@example.com",
                "password": "SecurePassword123"
            }
        }


class UserLogin(BaseModel):
    """Request model for user login."""
    email: EmailStr
    password: str


class TokenResponse(BaseModel):
    """Response model for successful login."""
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    user_id: str


class UserResponse(BaseModel):
    """Response model for user data."""
    id: str
    email: str


class MessageResponse(BaseModel):
    """Generic message response."""
    message: str
    user_id: Optional[str] = None


# === Helper Functions ===

def hash_password(password: str) -> str:
    """Hash a password using bcrypt."""
    return pwd_context.hash(password)


def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify a password against its hash."""
    return pwd_context.verify(plain_password, hashed_password)


def create_access_token(data: dict, expires_delta: timedelta = None) -> str:
    """Create a JWT access token."""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(hours=ACCESS_TOKEN_EXPIRE_HOURS)
    
    to_encode.update({"exp": expire})
    
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def decode_token(token: str) -> dict:
    """Decode and verify a JWT token."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None


# === API Endpoints ===

@router.post("/register", response_model=MessageResponse, status_code=status.HTTP_201_CREATED)
def register(user: UserRegister):
    """
    Register a new user.
    
    - **email**: Valid email address
    - **password**: Password (min 8 characters recommended)
    """
    # Check if email already exists
    if user.email.lower() in users_db:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # Validate password strength (basic)
    if len(user.password) < 6:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Password must be at least 6 characters"
        )
    
    # Create user
    import uuid
    user_id = str(uuid.uuid4())
    
    users_db[user.email.lower()] = {
        "id": user_id,
        "email": user.email.lower(),
        "password_hash": hash_password(user.password),
        "created_at": datetime.utcnow().isoformat()
    }
    
    return MessageResponse(
        message="User registered successfully",
        user_id=user_id
    )


@router.post("/login", response_model=TokenResponse)
def login(user: UserLogin):
    """
    Login and receive JWT token.
    
    - **email**: Registered email address
    - **password**: User's password
    """
    # Find user
    stored_user = users_db.get(user.email.lower())
    
    if not stored_user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Verify password
    if not verify_password(user.password, stored_user["password_hash"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Create token
    access_token = create_access_token(
        data={
            "sub": stored_user["id"],
            "email": stored_user["email"]
        }
    )
    
    return TokenResponse(
        access_token=access_token,
        token_type="bearer",
        expires_in=ACCESS_TOKEN_EXPIRE_HOURS * 3600,
        user_id=stored_user["id"]
    )


@router.get("/me", response_model=UserResponse)
def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """
    Get current authenticated user's information.
    
    Requires Bearer token in Authorization header.
    """
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token"
        )
    
    user_id = payload.get("sub")
    email = payload.get("email")
    
    # Verify user still exists
    stored_user = users_db.get(email)
    if not stored_user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found"
        )
    
    return UserResponse(id=user_id, email=email)


# === Dependency for Protected Routes ===

def get_current_user_id(credentials: HTTPAuthorizationCredentials = Depends(security)) -> str:
    """
    Dependency to get current user ID from token.
    
    Usage:
        @router.get("/protected")
        def protected_route(user_id: str = Depends(get_current_user_id)):
            ...
    """
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token"
        )
    
    return payload.get("sub")
```

### Step 3: Update Main App

Update file: `backend/main.py` - add these lines:

```python
# Add at the imports section
from api.auth import router as auth_router

# Add after the app definition (before the existing routes)
app.include_router(auth_router)
```

### Step 4: Test the Endpoints

```bash
cd backend
uvicorn main:app --reload

# In another terminal:

# 1. Test registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "password123"}'

# Expected: {"message":"User registered successfully","user_id":"..."}

# 2. Test login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "password123"}'

# Expected: {"access_token":"eyJ...","token_type":"bearer",...}

# 3. Test invalid login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "wrongpassword"}'

# Expected: {"detail":"Invalid email or password"}

# 4. Test duplicate registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "password123"}'

# Expected: {"detail":"Email already registered"}

# 5. Test /me endpoint (replace TOKEN with actual token from login)
curl http://localhost:8000/api/auth/me \
  -H "Authorization: Bearer TOKEN"

# Expected: {"id":"...","email":"test@test.com"}
```

### Step 5: Commit and PR

```bash
git add backend/api/ backend/main.py
git commit -m "AG-25: Create auth endpoints

- POST /api/auth/register - Create new user
- POST /api/auth/login - Return JWT token
- GET /api/auth/me - Get current user
- Password hashing with bcrypt
- JWT with 24h expiry"

git push origin feature/AG-25-auth-endpoints
```

**Jira:** Move AG-25 to "Done"

---

## 👨‍💻 Dev 3 (Marva): AG-29 - Frontend API Connection

### Step 1: Create Branch

```bash
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-29-api-connection
```

**Jira:** Move AG-29 to "In Progress"

### Step 2: Create API Service

Create file: `frontend/src/services/api.js`

```javascript
/**
 * API Service
 * 
 * Handles all HTTP requests to the backend API.
 */

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

/**
 * Register a new user
 */
export async function register(email, password) {
  const response = await fetch(`${API_URL}/api/auth/register`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ email, password }),
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.detail || 'Registration failed');
  }

  return data;
}

/**
 * Login user and store token
 */
export async function login(email, password) {
  const response = await fetch(`${API_URL}/api/auth/login`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ email, password }),
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.detail || 'Login failed');
  }

  // Store token and user info
  localStorage.setItem('token', data.access_token);
  localStorage.setItem('userId', data.user_id);

  return data;
}

/**
 * Logout user
 */
export function logout() {
  localStorage.removeItem('token');
  localStorage.removeItem('userId');
}

/**
 * Check if user is logged in
 */
export function isLoggedIn() {
  return !!localStorage.getItem('token');
}

/**
 * Get stored token
 */
export function getToken() {
  return localStorage.getItem('token');
}

/**
 * Get current user info
 */
export async function getCurrentUser() {
  const token = getToken();
  
  if (!token) {
    throw new Error('Not logged in');
  }

  const response = await fetch(`${API_URL}/api/auth/me`, {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });

  if (!response.ok) {
    throw new Error('Failed to get user info');
  }

  return response.json();
}
```

### Step 3: Update Login Page

Update file: `frontend/src/pages/Login.jsx`

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { login } from '../services/api';
import './Auth.css';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  const navigate = useNavigate();

  const validateForm = () => {
    setError('');
    
    if (!email.trim()) {
      setError('Email is required');
      return false;
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Please enter a valid email address');
      return false;
    }
    
    if (!password) {
      setError('Password is required');
      return false;
    }
    
    return true;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      await login(email, password);
      navigate('/dashboard');
    } catch (err) {
      setError(err.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>Welcome Back</h1>
        <p className="auth-subtitle">Sign in to continue</p>
        
        {error && <div className="error-message">{error}</div>}
        
        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              disabled={loading}
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Enter your password"
              disabled={loading}
            />
          </div>
          
          <button type="submit" className="auth-button" disabled={loading}>
            {loading ? 'Signing in...' : 'Sign In'}
          </button>
        </form>
        
        <p className="auth-footer">
          Don't have an account? <Link to="/register">Create one</Link>
        </p>
      </div>
    </div>
  );
}

export default Login;
```

### Step 4: Update Register Page

Update file: `frontend/src/pages/Register.jsx`

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { register } from '../services/api';
import './Auth.css';

function Register() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);
  
  const navigate = useNavigate();

  const validateForm = () => {
    setError('');
    
    if (!email.trim()) {
      setError('Email is required');
      return false;
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Please enter a valid email address');
      return false;
    }
    
    if (!password) {
      setError('Password is required');
      return false;
    }
    
    if (password.length < 8) {
      setError('Password must be at least 8 characters');
      return false;
    }
    
    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return false;
    }
    
    return true;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      await register(email, password);
      setSuccess(true);
      
      // Redirect to login after 2 seconds
      setTimeout(() => {
        navigate('/login');
      }, 2000);
    } catch (err) {
      setError(err.message || 'Registration failed');
    } finally {
      setLoading(false);
    }
  };

  if (success) {
    return (
      <div className="auth-container">
        <div className="auth-card">
          <h1>✓ Account Created!</h1>
          <p>Redirecting to login...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>Create Account</h1>
        <p className="auth-subtitle">Get started with Resume Agent</p>
        
        {error && <div className="error-message">{error}</div>}
        
        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              disabled={loading}
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Create a password"
              disabled={loading}
            />
            <small className="form-hint">At least 8 characters</small>
          </div>
          
          <div className="form-group">
            <label htmlFor="confirmPassword">Confirm Password</label>
            <input
              id="confirmPassword"
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              placeholder="Confirm your password"
              disabled={loading}
            />
          </div>
          
          <button type="submit" className="auth-button" disabled={loading}>
            {loading ? 'Creating account...' : 'Create Account'}
          </button>
        </form>
        
        <p className="auth-footer">
          Already have an account? <Link to="/login">Sign in</Link>
        </p>
      </div>
    </div>
  );
}

export default Register;
```

### Step 5: Test the Full Flow

```bash
# Terminal 1: Backend
cd backend && uvicorn main:app --reload

# Terminal 2: Frontend
cd frontend && npm run dev
```

### Step 6: Commit and PR

```bash
git add frontend/
git commit -m "AG-29: Connect frontend to auth API

- Create API service with login/register functions
- Store JWT token in localStorage
- Update Login page to call API
- Update Register page to call API
- Redirect on success"

git push origin feature/AG-29-api-connection
```

**Jira:** Move AG-29 to "Done"

---

## ✅ Day 7: Manual Verification

### Terminal Tests (API)

```bash
# Register new user
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "demo@example.com", "password": "Demo1234"}'

# Login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "demo@example.com", "password": "Demo1234"}'
```

### Browser Tests (Full User Flow)

1. **Open http://localhost:5173/register**
2. Fill in email, password, confirm password
3. Click "Create Account"
4. Should see success message
5. Should redirect to login

6. **On Login page:**
7. Enter same credentials
8. Click "Sign In"
9. Should redirect to Dashboard

10. **Open DevTools → Application → Local Storage**
11. Should see `token` key with JWT value

### What Must Work

| Test | Expected |
|------|----------|
| Register new user | Returns user_id |
| Register duplicate | Returns error |
| Login valid credentials | Returns token |
| Login invalid credentials | Returns error |
| Frontend register flow | Success message, redirect |
| Frontend login flow | Redirect to dashboard |
| Token stored | In localStorage |

---

## 📁 Folder Structure After Day 7

```
backend/api/
├── __init__.py
└── auth.py           ✓ NEW

frontend/src/
├── services/
│   └── api.js        ✓ NEW
└── pages/
    ├── Login.jsx     ✓ Updated
    └── Register.jsx  ✓ Updated
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-32: Login Page Component | Dev 3 | 3 | ✓ Done |
| AG-33: Register Page Component | Dev 3 | 3 | ✓ Done |
| AG-34: Auth API Service Module | Dev 1 | 3 | ✓ Done |
| AG-35: Protected Routes Setup | Dev 2 | 2 | ✓ Done |

**Total Story Points Completed:** 11  
**Dev 1:** 3 points | **Dev 2:** 2 points | **Dev 3:** 6 points

---

**← [Day 6](./day-06.md)** | **[Day 8: Agent API + Dashboard →](./day-08.md)**
