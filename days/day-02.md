# Day 2: First Agent Node

> **Date:** Sprint 1, Day 2  
> **Focus:** Build FIRST working LangGraph node + test workflow  
> **Vertical Slice:** First agent node works end-to-end

---

## 🎯 Today's Goal

By end of today:
- ✅ First LangGraph node (Job Requirements Extraction) working
- ✅ Test workflow validates the node runs correctly
- ✅ React project initialized with Vite
- ✅ Can test one complete agent node

---

## 📋 Morning: Add These Tasks to Jira Backlog

---

### Task AG-15: Job Requirements Extraction Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-15 |
| **Title** | Job Requirements Extraction Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Build the FIRST LangGraph node for the resume agent. This node uses GPT-4o-mini to extract structured requirements from a job description (skills, experience, keywords). This is everyone's introduction to building real agent nodes.

**What You'll Learn:**
- Creating LangGraph nodes (functions that transform state)
- Using OpenAI API for structured extraction
- Returning partial state updates from nodes
- Testing individual nodes

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/job_requirements.py`
- [ ] Node extracts: required_skills, preferred_skills, experience_years, key_keywords
- [ ] Returns dict that updates state's `job_requirements` field
- [ ] Includes example test with sample job description
- [ ] PR merged to main

**Dependencies:** AG-10 (State schema must be defined first)

**Related Tasks:** All developers learning LangGraph together

---

### Task AG-16: Test Workflow for Single Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-16 |
| **Title** | Test Workflow for Single Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a simple LangGraph workflow that runs just the Job Requirements node (AG-15). This tests that our first node works correctly and introduces the team to workflow compilation and invocation.

**What You'll Learn:**
- Building StateGraph workflows
- Adding nodes to graphs
- Setting entry points and edges
- Compiling and invoking workflows
- Testing agent outputs

**Acceptance Criteria:**
- [ ] File created: `backend/agent/test_workflow.py`
- [ ] Workflow includes only the job_requirements node
- [ ] Can invoke with test job description
- [ ] Prints extracted requirements in readable format
- [ ] Successfully runs end-to-end

**Dependencies:** AG-15 (Job requirements node must exist first)

**Related Tasks:** Testing what Dev 1 built

---

### Task AG-17: React Project Setup

| Field | Value |
|-------|-------|
| **Task ID** | AG-17 |
| **Title** | React Project Setup |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Initialize React project using Vite. Set up React Router with routes for all pages: Home, Login, Register, Dashboard, and Results. Create basic page components (can be placeholder content for now).

**What You'll Learn:**
- Creating React apps with Vite
- React Router v6 setup
- Project structure for React apps
- Component-based architecture

**Acceptance Criteria:**
- [ ] React app created with Vite (react template)
- [ ] React Router DOM installed and configured
- [ ] Routes defined for: `/`, `/login`, `/register`, `/dashboard`, `/results/:id`
- [ ] Basic page components created (can show "Page Name" for now)
- [ ] App runs without errors on `npm run dev`
- [ ] `.env.example` file created for API URL
- [ ] PR merged to main

**Dependencies:** AG-13 (Frontend CI/CD must be set up first)

---

## 🕘 9:00 AM - Daily Standup

**Standup Questions (each person answers):**
1. What did you complete yesterday?
2. What will you work on today?
3. Any blockers?

**Expected Answers:**
- **Dev 1:** Yesterday: LangGraph tutorial. Today: AG-8 Agent State. No blockers.
- **Dev 2:** Yesterday: Supabase setup. Today: AG-18 FastAPI structure. No blockers.
- **Dev 3:** Yesterday: CI/CD pipelines. Today: AG-81 React setup. No blockers.

**After Standup:**
- Add today's tasks to Jira
- Move tasks to "To Do"

---

## 👨‍💻 Dev 1 (Shabas): AG-8 - Agent State Schema

### What to Learn First (10 minutes)

**Read:** https://docs.python.org/3/library/typing.html#typing.TypedDict

**Key Concepts:**
- TypedDict creates type-safe dictionaries
- Each field has a declared type
- Optional fields use `Optional[Type]`
- List fields use `List[Type]`

**Example:**
```python
from typing import TypedDict, Optional

class Person(TypedDict):
    name: str
    age: int
    email: Optional[str]
```

---

### Step-by-Step Instructions

#### Step 1: Update Your Local Repository

```bash
cd resume-agent

# Make sure you have the latest code
git checkout dev-shabas
git pull origin main
```

#### Step 2: Create Feature Branch

```bash
git checkout -b feature/AG-8-agent-state-schema
```

**Jira Update:** Move AG-8 to "In Progress"

#### Step 3: Create Folder Structure

```bash
# Create agent directory with nodes subdirectory
mkdir -p backend/agent/nodes

# Create __init__.py files (makes them Python packages)
touch backend/agent/__init__.py
touch backend/agent/nodes/__init__.py
```

#### Step 4: Create State File

Create file: `backend/agent/state.py`

```python
"""
ResumeAgentState - The State Definition for our Resume Optimization Agent

This TypedDict defines ALL data that flows through the agent workflow.
Think of it as the "memory" of the agent - every node can read from it
and write updates to it.

Key sections:
- Identifiers: run_id, user_id
- Inputs: what the user provides (JD, resume)
- Extracted Data: what we parse from inputs
- Scoring: ATS scores before/after
- Planning: what changes to make
- Outputs: modified resume, cover letter
- Control: iteration count, status
"""
from typing import TypedDict, List, Dict, Optional


class ResumeAgentState(TypedDict):
    """
    The complete state for a resume optimization run.
    
    Every node in the workflow receives this state and can return
    a partial dict to update specific fields.
    """
    
    # === IDENTIFIERS ===
    # These help us track and store the run
    run_id: str              # Unique identifier for this optimization run
    user_id: str             # ID of the user who started this run
    
    # === USER INPUTS ===
    # What the user provides to start the optimization
    job_description: str     # The full job posting text
    original_resume: str     # User's original resume content
    
    # === EXTRACTED DATA ===
    # Parsed and structured data from the inputs
    job_requirements: Optional[Dict]   # Extracted from JD: skills, keywords, experience
    resume_analysis: Optional[Dict]    # Analysis of current resume: strengths, gaps
    
    # === SCORING ===
    # ATS (Applicant Tracking System) scores
    ats_score_before: Optional[float]  # Initial score (0-100) before any changes
    ats_score_after: Optional[float]   # Final score after modifications
    improvement_delta: Optional[float] # How much the score improved
    score_history: List[float]         # Track scores across iterations
    
    # === PLANNING & MODIFICATION ===
    # What changes to make and what was changed
    improvement_plan: Optional[Dict]   # Planned changes with priorities
    modified_resume: Optional[str]     # The resume after applying changes
    applied_changes: List[Dict]        # Log of all changes made
    
    # === DECISION TRACKING ===
    # This is what makes it "agentic" - we log all decisions
    decision_log: List[Dict]           # Each decision with reasoning
    
    # === OUTPUTS ===
    # Final deliverables for the user
    cover_letter: Optional[str]        # Generated cover letter
    
    # === CONTROL PARAMETERS ===
    # Control the agent's behavior
    iteration_count: int               # Current iteration (starts at 0)
    max_iterations: int                # Maximum iterations allowed (default: 3)
    final_status: str                  # Status: pending, running, completed, failed


def create_initial_state(
    run_id: str,
    user_id: str,
    job_description: str,
    original_resume: str,
    max_iterations: int = 3
) -> ResumeAgentState:
    """
    Factory function to create a new initial state.
    
    Use this instead of manually creating the dict to ensure
    all fields are properly initialized.
    """
    return ResumeAgentState(
        # Identifiers
        run_id=run_id,
        user_id=user_id,
        
        # User inputs
        job_description=job_description,
        original_resume=original_resume,
        
        # Extracted data (filled by nodes)
        job_requirements=None,
        resume_analysis=None,
        
        # Scoring (filled by scoring nodes)
        ats_score_before=None,
        ats_score_after=None,
        improvement_delta=None,
        score_history=[],
        
        # Planning (filled by planning node)
        improvement_plan=None,
        modified_resume=None,
        applied_changes=[],
        
        # Decision tracking
        decision_log=[],
        
        # Outputs
        cover_letter=None,
        
        # Control
        iteration_count=0,
        max_iterations=max_iterations,
        final_status="pending"
    )
```

#### Step 5: Create Test File

Create file: `backend/agent/test_state.py`

```python
"""
Tests for the ResumeAgentState definition.

Run with: python test_state.py
"""
from state import ResumeAgentState, create_initial_state


def test_create_state_manually():
    """Test that we can create a state dict manually."""
    state: ResumeAgentState = {
        "run_id": "test-run-123",
        "user_id": "user-456",
        "job_description": "Looking for a Python developer with Django experience...",
        "original_resume": "John Doe\nSoftware Engineer\n3 years experience...",
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
    
    # Verify fields
    assert state["run_id"] == "test-run-123"
    assert state["iteration_count"] == 0
    assert state["final_status"] == "pending"
    assert state["score_history"] == []
    
    print("✓ Manual state creation test passed!")


def test_create_initial_state_factory():
    """Test the factory function for creating initial state."""
    state = create_initial_state(
        run_id="factory-run-789",
        user_id="user-abc",
        job_description="We need a React developer...",
        original_resume="Jane Smith\nFrontend Developer..."
    )
    
    # Verify required fields
    assert state["run_id"] == "factory-run-789"
    assert state["user_id"] == "user-abc"
    assert state["job_description"].startswith("We need")
    
    # Verify defaults
    assert state["max_iterations"] == 3
    assert state["iteration_count"] == 0
    assert state["final_status"] == "pending"
    
    # Verify optional fields are None
    assert state["job_requirements"] is None
    assert state["ats_score_before"] is None
    assert state["cover_letter"] is None
    
    print("✓ Factory function test passed!")


def test_state_update_simulation():
    """Test that we can update state like LangGraph nodes do."""
    state = create_initial_state(
        run_id="update-test",
        user_id="user-xyz",
        job_description="Python developer needed",
        original_resume="My resume content"
    )
    
    # Simulate what a scoring node would return
    score_update = {
        "ats_score_before": 45.5,
        "score_history": [45.5]
    }
    
    # Apply update (this is what LangGraph does internally)
    updated_state = {**state, **score_update}
    
    # Verify update applied
    assert updated_state["ats_score_before"] == 45.5
    assert updated_state["score_history"] == [45.5]
    
    # Verify other fields unchanged
    assert updated_state["run_id"] == "update-test"
    assert updated_state["iteration_count"] == 0
    
    print("✓ State update simulation test passed!")


if __name__ == "__main__":
    test_create_state_manually()
    test_create_initial_state_factory()
    test_state_update_simulation()
    print("\n🎉 All state tests passed!")
```

#### Step 6: Run Tests

```bash
cd backend/agent
python test_state.py
```

**Expected Output:**
```
✓ Manual state creation test passed!
✓ Factory function test passed!
✓ State update simulation test passed!

🎉 All state tests passed!
```

#### Step 7: Check What Files You Created

```bash
cd ../..  # Back to resume-agent root
git status
```

Should show:
```
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        backend/agent/
```

#### Step 8: Add and Commit

```bash
git add backend/agent/

git commit -m "AG-8: Define ResumeAgentState TypedDict with all fields

- Create state.py with complete state definition
- Add all required fields: identifiers, inputs, scoring, planning, outputs
- Add factory function create_initial_state()
- Add comprehensive test file
- All tests passing"
```

#### Step 9: Push to Feature Branch

```bash
git push origin feature/AG-8-agent-state-schema
```

**Jira Update:** Move AG-8 to "In Review"

#### Step 10: Open Pull Request

1. Go to GitHub repository
2. Click "Compare & pull request"
3. Fill in details:

**Title:**
```
AG-8: Define ResumeAgentState TypedDict with all fields
```

**Description:**
```markdown
## Summary
Define the complete state schema for the Resume Optimization Agent.

## Jira Task
AG-8: Agent State Schema

## What This State Contains
- **Identifiers:** run_id, user_id
- **Inputs:** job_description, original_resume
- **Extracted Data:** job_requirements, resume_analysis
- **Scoring:** ats_score_before, ats_score_after, score_history
- **Planning:** improvement_plan, modified_resume, applied_changes
- **Outputs:** cover_letter
- **Control:** iteration_count, max_iterations, final_status

## Files Added
- `backend/agent/__init__.py` - package marker
- `backend/agent/state.py` - main state definition
- `backend/agent/test_state.py` - test file
- `backend/agent/nodes/__init__.py` - nodes package marker

## Testing
```bash
cd backend/agent
python test_state.py
# All 3 tests pass
```

## Checklist
- [x] TypedDict properly defined
- [x] All fields have correct types
- [x] Factory function works
- [x] Tests pass
```

4. Request review from Dev 2
5. After approval, merge
6. Delete branch

#### Step 11: Cleanup After Merge

```bash
git checkout dev-shabas
git pull origin main
git branch -d feature/AG-8-agent-state-schema
```

**Jira Update:** Move AG-8 to "Done"

---

## 👨‍💻 Dev 2 (Sinan): AG-18 - FastAPI Project Structure

### What to Learn First (30 minutes)

**Read:**
- FastAPI First Steps: https://fastapi.tiangolo.com/tutorial/first-steps/
- SQLAlchemy Quick Start: https://docs.sqlalchemy.org/en/20/orm/quickstart.html

**Key Concepts:**
- FastAPI uses routers to organize endpoints
- SQLAlchemy connects Python to databases
- Environment variables store secrets safely

---

### Step-by-Step Instructions

#### Step 1: Update Your Local Repository

```bash
cd resume-agent
git checkout sinan-Dev
git pull origin main
```

#### Step 2: Create Feature Branch

```bash
git checkout -b feature/AG-18-fastapi-structure
```

**Jira Update:** Move AG-18 to "In Progress"

#### Step 3: Create Folder Structure

```bash
mkdir -p backend/api
mkdir -p backend/database

touch backend/api/__init__.py
touch backend/database/__init__.py
```

#### Step 4: Create Database Connection

Create file: `backend/database/connection.py`

```python
"""
Database Connection - Connects to Supabase PostgreSQL

This module handles the database session management.
Uses SQLAlchemy for ORM capabilities.
"""
import os
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from sqlalchemy.ext.declarative import declarative_base

# Get database URL from environment
DATABASE_URL = os.getenv("DATABASE_URL")

# Create engine only if DATABASE_URL is set
if DATABASE_URL:
    # SQLAlchemy needs postgresql:// but Supabase gives postgres://
    if DATABASE_URL.startswith("postgres://"):
        DATABASE_URL = DATABASE_URL.replace("postgres://", "postgresql://", 1)
    
    engine = create_engine(
        DATABASE_URL,
        pool_pre_ping=True,  # Verify connections before using
        pool_size=5,         # Max connections to keep open
        max_overflow=10      # Extra connections when needed
    )
    SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
else:
    engine = None
    SessionLocal = None

# Base class for ORM models
Base = declarative_base()


def get_db() -> Session:
    """
    Dependency for FastAPI endpoints.
    
    Usage:
        @app.get("/items")
        def get_items(db: Session = Depends(get_db)):
            ...
    """
    if SessionLocal is None:
        raise RuntimeError("Database not configured. Set DATABASE_URL environment variable.")
    
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


def check_database_connection() -> bool:
    """Test if database connection works."""
    if engine is None:
        return False
    
    try:
        with engine.connect() as conn:
            conn.execute("SELECT 1")
        return True
    except Exception:
        return False
```

#### Step 5: Update Main Application

Update file: `backend/main.py`

```python
"""
Resume Agent API - Main FastAPI Application

This is the entry point for the backend API.
Run with: uvicorn main:app --reload
"""
import os
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Create FastAPI app
app = FastAPI(
    title="Resume Agent API",
    description="AI-powered resume optimization service",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# Configure CORS
# In production, replace "*" with actual frontend URLs
allowed_origins = os.getenv("ALLOWED_ORIGINS", "*").split(",")

app.add_middleware(
    CORSMiddleware,
    allow_origins=allowed_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/")
def root():
    """Root endpoint - API information."""
    return {
        "service": "Resume Agent API",
        "version": "1.0.0",
        "status": "running",
        "documentation": {
            "swagger": "/docs",
            "redoc": "/redoc"
        }
    }


@app.get("/health")
def health():
    """Health check endpoint for monitoring and CI/CD."""
    from database.connection import check_database_connection
    
    db_status = check_database_connection()
    
    return {
        "status": "ok" if db_status else "degraded",
        "database": "connected" if db_status else "disconnected"
    }


# Import and include routers as we create them
# from api.auth import router as auth_router
# from api.agent import router as agent_router
# app.include_router(auth_router)
# app.include_router(agent_router)
```

#### Step 6: Create .env.example File

Create file: `backend/.env.example`

```bash
# Database Connection (from Supabase)
# Format: postgresql://postgres:[PASSWORD]@db.[PROJECT].supabase.co:5432/postgres
DATABASE_URL=postgresql://postgres:your-password@db.xxxx.supabase.co:5432/postgres

# Security
# Generate with: python -c "import secrets; print(secrets.token_urlsafe(32))"
SECRET_KEY=your-secret-key-change-in-production

# LLM API Keys
OPENAI_API_KEY=sk-your-openai-key-here

# CORS - Allowed frontend origins (comma-separated)
ALLOWED_ORIGINS=http://localhost:5173,http://localhost:3000

# Environment
ENVIRONMENT=development
```

#### Step 7: Update requirements.txt

Update file: `backend/requirements.txt`

```
# Web Framework
fastapi==0.109.0
uvicorn[standard]==0.27.0

# Database
sqlalchemy==2.0.25
psycopg2-binary==2.9.9

# Environment & Config
python-dotenv==1.0.0
pydantic==2.5.3
pydantic-settings==2.1.0

# Authentication
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4

# LangGraph & LLM
langgraph==0.0.20
langchain==0.1.0
langchain-openai==0.0.5

# Utilities
httpx==0.26.0
```

#### Step 8: Test the Setup

```bash
cd backend

# Create a local .env file (copy from example)
cp .env.example .env

# Edit .env and add your Supabase DATABASE_URL
# (Use the connection string you saved yesterday)

# Install dependencies
pip install -r requirements.txt

# Run the server
uvicorn main:app --reload
```

Test in another terminal:
```bash
# Test health endpoint
curl http://localhost:8000/health
# Expected: {"status":"ok","database":"connected"} or {"status":"degraded","database":"disconnected"}

# Test root endpoint
curl http://localhost:8000/
# Expected: JSON with service info
```

#### Step 9: Commit and Push

```bash
cd ..  # Back to resume-agent root
git add .
git commit -m "AG-18: Setup FastAPI project structure with database connection

- Create api/ and database/ directories
- Add database connection with SQLAlchemy
- Update main.py with CORS and improved structure
- Create .env.example with all required variables
- Update requirements.txt with all dependencies"

git push origin feature/AG-18-fastapi-structure
```

**Jira Update:** Move AG-18 to "In Review"

#### Step 10: Open Pull Request

Similar to previous PRs - request review, merge, cleanup.

**Jira Update:** After merge, move AG-18 to "Done"

---

## 👨‍💻 Dev 3 (Marva): AG-81 - React Project Setup

### What to Learn First (30 minutes)

**Read:**
- Vite Getting Started: https://vitejs.dev/guide/
- React Router Tutorial: https://reactrouter.com/en/main/start/tutorial

**Key Concepts:**
- Vite is a fast build tool for React
- React Router handles page navigation
- Routes map URLs to components

---

### Step-by-Step Instructions

#### Step 1: Update Your Local Repository

```bash
cd resume-agent
git checkout dev-marva
git pull origin main
```

#### Step 2: Create Feature Branch

```bash
git checkout -b feature/AG-81-react-setup
```

**Jira Update:** Move AG-81 to "In Progress"

#### Step 3: Create React App with Vite

```bash
# Create React app in frontend folder
npm create vite@latest frontend -- --template react

# Navigate to frontend
cd frontend

# Install base dependencies
npm install

# Install additional dependencies
npm install react-router-dom axios
```

#### Step 4: Set Up Routing

Update file: `frontend/src/App.jsx`

```jsx
/**
 * Main App Component
 * 
 * This is the root component that sets up routing for the entire application.
 * Each route renders a different page component.
 */
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

// Page Components (placeholders for now - will build properly later)
function Home() {
  return (
    <div style={{ padding: '40px', textAlign: 'center' }}>
      <h1>Resume Optimization Agent</h1>
      <p>AI-powered resume improvement</p>
      <div style={{ marginTop: '20px' }}>
        <a href="/login" style={{ marginRight: '10px' }}>Login</a>
        <a href="/register">Register</a>
      </div>
    </div>
  );
}

function Login() {
  return (
    <div style={{ padding: '40px', maxWidth: '400px', margin: '0 auto' }}>
      <h1>Login</h1>
      <p>Login page - will add form later</p>
      <a href="/register">Don't have an account? Register</a>
    </div>
  );
}

function Register() {
  return (
    <div style={{ padding: '40px', maxWidth: '400px', margin: '0 auto' }}>
      <h1>Register</h1>
      <p>Registration page - will add form later</p>
      <a href="/login">Already have an account? Login</a>
    </div>
  );
}

function Dashboard() {
  return (
    <div style={{ padding: '40px' }}>
      <h1>Dashboard</h1>
      <p>Upload your resume and job description here</p>
    </div>
  );
}

function Results() {
  return (
    <div style={{ padding: '40px' }}>
      <h1>Optimization Results</h1>
      <p>Your optimized resume will appear here</p>
    </div>
  );
}

function NotFound() {
  return (
    <div style={{ padding: '40px', textAlign: 'center' }}>
      <h1>404 - Page Not Found</h1>
      <a href="/">Go Home</a>
    </div>
  );
}

/**
 * Main App with Routing
 */
function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public Routes */}
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
        
        {/* Protected Routes (will add auth check later) */}
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/results/:id" element={<Results />} />
        
        {/* 404 Fallback */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

#### Step 5: Create Environment Example

Create file: `frontend/.env.example`

```bash
# API URL - points to FastAPI backend
VITE_API_URL=http://localhost:8000

# Environment
VITE_ENV=development
```

#### Step 6: Update CSS for Basic Styling

Update file: `frontend/src/index.css`

```css
/* Global Styles */
:root {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color: #213547;
  background-color: #ffffff;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  min-height: 100vh;
}

a {
  color: #646cff;
  text-decoration: none;
}

a:hover {
  color: #535bf2;
}

h1 {
  font-size: 2.5em;
  line-height: 1.1;
  margin-bottom: 0.5em;
}

button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  background-color: #1a1a1a;
  color: white;
  cursor: pointer;
  transition: border-color 0.25s;
}

button:hover {
  border-color: #646cff;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

input, textarea {
  border-radius: 8px;
  border: 1px solid #ccc;
  padding: 0.6em 1em;
  font-size: 1em;
  font-family: inherit;
  width: 100%;
}

input:focus, textarea:focus {
  outline: none;
  border-color: #646cff;
}
```

#### Step 7: Test the Application

```bash
# Make sure you're in frontend directory
cd frontend

# Run development server
npm run dev
```

Open browser and test each route:
- http://localhost:5173 → Home page
- http://localhost:5173/login → Login page
- http://localhost:5173/register → Register page
- http://localhost:5173/dashboard → Dashboard page
- http://localhost:5173/results/123 → Results page
- http://localhost:5173/random → 404 page

#### Step 8: Commit and Push

```bash
cd ..  # Back to resume-agent root

git add frontend/

git commit -m "AG-81: Initialize React project with Vite and routing

- Create React app using Vite
- Install react-router-dom and axios
- Set up routes for all pages: /, /login, /register, /dashboard, /results/:id
- Add basic placeholder components for each page
- Add global CSS styling
- Create .env.example for API URL configuration"

git push origin feature/AG-81-react-setup
```

**Jira Update:** Move AG-81 to "In Review"

#### Step 9: Open PR, Merge, Cleanup

Similar to previous PRs.

**Jira Update:** After merge, move AG-81 to "Done"

---

## ✅ End of Day 2: Manual Verification

### Terminal Tests - Backend

```bash
cd resume-agent/backend

# Install dependencies
pip install -r requirements.txt

# Run backend
uvicorn main:app --reload

# In another terminal:
# 1. Test health endpoint
curl http://localhost:8000/health
# Expected: {"status":"ok","database":"connected"} or {"status":"degraded"}

# 2. Test Swagger docs
curl -s http://localhost:8000/docs | grep -o "<title>.*</title>"
# Expected: <title>Resume Agent API - Swagger UI</title>
```

### Terminal Tests - Agent State

```bash
cd resume-agent/backend/agent

# Run state tests
python test_state.py
# Expected: All 3 tests pass
```

### Terminal Tests - Frontend

```bash
cd resume-agent/frontend

# Run development server
npm run dev

# In another terminal, verify it's running:
curl -s http://localhost:5173 | head -5
# Expected: HTML content
```

### Browser Tests

1. **Backend API:**
   - Open http://localhost:8000/docs
   - Should see Swagger UI
   - Test GET /health endpoint

2. **Frontend Routing:**
   - Open http://localhost:5173
   - Click "Login" link → Should go to /login
   - Click "Register" link → Should go to /register
   - Manually go to /dashboard → Should show Dashboard
   - Manually go to /results/test-123 → Should show Results
   - Go to /nonexistent → Should show 404 page

### Folder Structure Verification

```bash
tree resume-agent -I node_modules
```

Expected:
```
resume-agent/
├── .github/workflows/
├── backend/
│   ├── main.py             ✓ Updated
│   ├── requirements.txt    ✓ Updated
│   ├── .env.example        ✓ New
│   ├── api/
│   │   └── __init__.py     ✓ New
│   ├── agent/
│   │   ├── __init__.py     ✓ New
│   │   ├── state.py        ✓ New
│   │   ├── test_state.py   ✓ New
│   │   └── nodes/
│   │       └── __init__.py ✓ New
│   └── database/
│       ├── __init__.py     ✓ New
│       └── connection.py   ✓ New
├── frontend/
│   ├── package.json        ✓ New
│   ├── vite.config.js      ✓ New
│   ├── .env.example        ✓ New
│   ├── index.html          ✓ New
│   └── src/
│       ├── main.jsx        ✓ New
│       ├── App.jsx         ✓ New (with routing)
│       └── index.css       ✓ Updated
└── README.md
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-15: Job Requirements Extraction Node | Dev 1 | 5 | ✓ Done |
| AG-16: Test Workflow for Single Node | Dev 2 | 3 | ✓ Done |
| AG-17: React Setup | Dev 3 | 5 | ✓ Done |

**Total Story Points Completed:** 13  
**Dev 1:** 5 points | **Dev 2:** 3 points | **Dev 3:** 5 points

---

**← [Day 1: Setup](./day-01.md)** | **[Day 3: First LangGraph Nodes →](./day-03.md)**
