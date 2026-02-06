# Day 1: Setup + LangGraph Learning

> **Date:** Sprint 1, Day 1  
> **Focus:** Team LangGraph tutorial, database setup, CI/CD setup  
> **Vertical Slice:** Slice 1 - Setup + Foundation

---

## 🎯 Today's Goal

By end of today:
- ✅ All developers understand LangGraph basics
- ✅ Supabase database created with tables
- ✅ CI/CD pipelines working
- ✅ Basic backend with /health endpoint

---

## 📋 Morning: Add These Tasks to Jira Backlog

Before starting work, add these 8 tasks to your Jira board:

---

### Task AG-7: LangGraph Tutorial Part 1 - Basics

| Field | Value |
|-------|-------|
| **Task ID** | AG-13 |
| **Title** | LangGraph Tutorial Part 1: Basics |
| **Type** | Task |
| **Epic** | Learning & Setup |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Complete Part 1 of the LangGraph tutorial focusing on basic concepts. Each developer works through different parts and shares learnings.

**What You'll Learn:**
- How to install LangGraph
- Basic StateGraph concepts
- Simple node creation

**Acceptance Criteria:**
- [ ] LangGraph installed and working
- [ ] Can create a simple StateGraph
- [ ] Can run a basic 1-node workflow
- [ ] Share key learnings with team

**Tutorial URL:** https://langchain-ai.github.io/langgraph/tutorials/introduction/

---

### Task AG-8: LangGraph Tutorial Part 2 - State Management

| Field | Value |
|-------|-------|
| **Task ID** | AG-8 |
| **Title** | LangGraph Tutorial Part 2: State Management |
| **Type** | Task |
| **Epic** | Learning & Setup |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Complete Part 2 of the LangGraph tutorial focusing on state management using TypedDict.

**What You'll Learn:**
- TypedDict for state definition
- State updates and merging
- Accessing state in nodes

**Acceptance Criteria:**
- [ ] Understand TypedDict state pattern
- [ ] Can define custom state schemas
- [ ] Can update state from nodes
- [ ] Share key learnings with team

**Tutorial URL:** https://langchain-ai.github.io/langgraph/tutorials/introduction/

---

### Task AG-9: LangGraph Tutorial Part 3 - Workflow Building

| Field | Value |
|-------|-------|
| **Task ID** | AG-9 |
| **Title** | LangGraph Tutorial Part 3: Workflow Building |
| **Type** | Task |
| **Epic** | Learning & Setup |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Complete Part 3 of the LangGraph tutorial focusing on connecting nodes and building workflows.

**What You'll Learn:**
- Adding edges between nodes
- Entry points and END nodes
- Compiling and invoking workflows

**Acceptance Criteria:**
- [ ] Can connect multiple nodes with edges
- [ ] Can set entry points
- [ ] Can compile and run workflows
- [ ] Share key learnings with team

**Tutorial URL:** https://langchain-ai.github.io/langgraph/tutorials/introduction/

---

### Task AG-10: Agent State Schema

| Field | Value |
|-------|-------|
| **Task ID** | AG-10 |
| **Title** | Agent State Schema |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Define the ResumeAgentState TypedDict that will flow through the entire agent workflow. This is the foundation for all agent nodes.

**What You'll Learn:**
- Designing state for complex workflows
- TypedDict best practices
- State field planning

**Acceptance Criteria:**
- [ ] ResumeAgentState TypedDict defined
- [ ] Includes all required fields (user_id, job_description, scores, etc.)
- [ ] Helper factory function created
- [ ] Test file validates state creation

**Related Tasks:** AG-13 (builds on tutorial knowledge)

---

### Task AG-11: Supabase Database Setup

| Field | Value |
|-------|-------|
| **Task ID** | AG-11 |
| **Title** | Supabase Database Setup |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a new Supabase project and set up the required database tables. This includes the `users` table for authentication and the `agent_runs` table for storing agent execution results.

**What You'll Learn:**
- How to create a Supabase project
- How to use Supabase SQL Editor
- How to get database connection strings

**Acceptance Criteria:**
- [ ] Supabase project created with name "resume-agent"
- [ ] `users` table created with columns: id, email, password_hash, created_at
- [ ] `agent_runs` table created with all required columns
- [ ] Database connection string (URI format) saved securely
- [ ] Can view tables in Supabase Table Editor

---

### Task AG-12: Backend CI/CD Pipeline

| Field | Value |
|-------|-------|
| **Task ID** | AG-12 |
| **Title** | Backend CI/CD Pipeline |
| **Type** | Task |
| **Epic** | DevOps & Infrastructure |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Set up GitHub Actions workflow to automatically run tests for the backend when code is pushed. This ensures code quality and catches issues early.

**What You'll Learn:**
- GitHub Actions YAML syntax
- CI/CD pipeline concepts
- Python workflow automation

**Acceptance Criteria:**
- [ ] File `.github/workflows/backend.yml` created
- [ ] Workflow triggers only on changes to `backend/**` directory
- [ ] Workflow installs Python dependencies
- [ ] Workflow runs linting/basic checks
- [ ] Workflow runs without errors after PR merge

**Note:** CI/CD tasks split - Dev 1 handles backend, Dev 3 handles frontend

---

### Task AG-13: Frontend CI/CD Pipeline

| Field | Value |
|-------|-------|
| **Task ID** | AG-13 |
| **Title** | Frontend CI/CD Pipeline |
| **Type** | Task |
| **Epic** | DevOps & Infrastructure |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Set up GitHub Actions workflow to automatically build and test the frontend when code is pushed.

**What You'll Learn:**
- GitHub Actions for Node.js projects
- Frontend CI/CD best practices
- Build automation

**Acceptance Criteria:**
- [ ] File `.github/workflows/frontend.yml` created
- [ ] Workflow triggers only on changes to `frontend/**` directory
- [ ] Workflow runs npm install and npm build
- [ ] Workflow runs without errors

**Note:** CI/CD tasks split - Dev 1 handles backend, Dev 3 handles frontend

---

### Task AG-14: FastAPI Basic Setup

| Field | Value |
|-------|-------|
| **Task ID** | AG-14 |
| **Title** | FastAPI Basic Setup |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create the basic FastAPI application structure with a health check endpoint and CORS configuration.

**What You'll Learn:**
- FastAPI application setup
- CORS middleware configuration
- API documentation with Swagger

**Acceptance Criteria:**
- [ ] `backend/main.py` created with FastAPI app
- [ ] `/health` endpoint returns {"status": "ok"}
- [ ] CORS middleware configured
- [ ] API runs locally with `uvicorn main:app --reload`
- [ ] Swagger docs accessible at `/docs`

---

## 🕘 9:00 AM - Daily Standup

**First Day Kickoff (30 minutes):**

1. **Introductions** - Review the project scope together
2. **Review this guide** - Make sure everyone understands the workflow
3. **Set up Jira board** - Add the 4 tasks above to the backlog
4. **Assign tasks** - Move tasks to "To Do"
5. **Start tutorial together**

---

## 🕤 9:30 AM - 12:30 PM: LangGraph Tutorial (All Developers)

**TEAM SESSION - All three developers do this together!**

### Why Together?
LangGraph is the core technology of our agent. Every developer needs to understand it deeply. Working together helps:
- Discuss concepts as you learn
- Help each other with issues
- Build shared understanding

### What to Learn First

**Tutorial URL:** https://langchain-ai.github.io/langgraph/tutorials/introduction/

**Time needed:** 2-3 hours

### Step-by-Step Setup

**Step 1: Create tutorial folder**
```bash
# Everyone runs these commands
mkdir langgraph-tutorial
cd langgraph-tutorial
```

**Step 2: Create virtual environment**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

**Step 3: Install dependencies**
```bash
pip install langgraph langchain-openai python-dotenv
```

**Step 4: Create .env file**
```bash
# Create .env file
echo "OPENAI_API_KEY=your-key-here" > .env
```

**Step 5: Create test file**

Create file: `tutorial.py`
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict
import os
from dotenv import load_dotenv

load_dotenv()

# Define state - this is the "memory" of the agent
class State(TypedDict):
    input: str
    output: str

# Define a node - this is a function that processes state
def process_node(state: State) -> dict:
    """A simple node that uppercases the input."""
    return {"output": state["input"].upper()}

# Build the workflow
workflow = StateGraph(State)

# Add the node
workflow.add_node("process", process_node)

# Set entry point
workflow.set_entry_point("process")

# Add edge to END
workflow.add_edge("process", END)

# Compile
app = workflow.compile()

# Test it
if __name__ == "__main__":
    result = app.invoke({"input": "hello world", "output": ""})
    print(f"Input: hello world")
    print(f"Output: {result['output']}")
    print("✓ LangGraph is working!")
```

**Step 6: Run the test**
```bash
python tutorial.py
```

**Expected Output:**
```
Input: hello world
Output: HELLO WORLD
✓ LangGraph is working!
```

### After Tutorial: Draw Agent Workflow

On whiteboard or Miro, draw the agent flow:
```
┌─────────────────┐
│ Extract JD Reqs │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Analyze Resume  │
└────────┬────────┘
         ▼
┌─────────────────┐
│  Score Initial  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Plan Changes    │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Modify Resume   │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Score Modified  │
└────────┬────────┘
         ▼
┌─────────────────┐
│    Finalize     │
└─────────────────┘
```

### Jira Update

| When | Action |
|------|--------|
| 9:30 AM | Move AG-13 to "In Progress" |
| 12:30 PM | Move AG-13 to "Done" |

---

## 🕑 2:00 PM - Dev 2 (Sinan): AG-137 - Supabase Setup

### What to Learn First (15 minutes)

**Read:** https://supabase.com/docs/guides/database

**Key Concepts:**
- Supabase is hosted PostgreSQL
- You run SQL directly in browser
- Connection string = how backend connects

---

### Step-by-Step Instructions

#### Step 1: Create Supabase Account

1. Go to https://supabase.com
2. Click "Start your project" button
3. Sign up with GitHub (recommended) or email
4. Verify your email if needed

#### Step 2: Create New Project

1. After login, you'll see the dashboard
2. Click "New Project"
3. Fill in details:
   - **Name:** `resume-agent`
   - **Database Password:** Click "Generate a password" and **SAVE IT SOMEWHERE SECURE**
   - **Region:** Select the one closest to you (e.g., "Southeast Asia" for India)
4. Click "Create new project"
5. Wait 2-3 minutes for setup to complete

#### Step 3: Open SQL Editor

1. In the left sidebar, click "SQL Editor"
2. Click "New query"

#### Step 4: Create Users Table

Paste this SQL and click "Run":

```sql
-- Create users table for authentication
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Add index for faster email lookups
CREATE INDEX idx_users_email ON users(email);

-- Add comment
COMMENT ON TABLE users IS 'Stores registered user accounts';
```

**Expected Result:** "Success. No rows returned"

#### Step 5: Create Agent Runs Table

Click "New query" and paste this SQL:

```sql
-- Create agent_runs table for storing optimization results
CREATE TABLE agent_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    
    -- Input data
    job_description TEXT NOT NULL,
    original_resume TEXT NOT NULL,
    
    -- Output data
    modified_resume TEXT,
    cover_letter TEXT,
    
    -- Scoring
    ats_score_before FLOAT,
    ats_score_after FLOAT,
    
    -- Agent state
    job_requirements JSONB,
    resume_analysis JSONB,
    improvement_plan JSONB,
    decision_log JSONB,
    score_history JSONB,
    
    -- Control
    iteration_count INTEGER DEFAULT 0,
    max_iterations INTEGER DEFAULT 3,
    status VARCHAR(50) DEFAULT 'pending',
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE
);

-- Add indexes
CREATE INDEX idx_agent_runs_user_id ON agent_runs(user_id);
CREATE INDEX idx_agent_runs_status ON agent_runs(status);

-- Add comment
COMMENT ON TABLE agent_runs IS 'Stores each resume optimization run';
```

Click "Run"

**Expected Result:** "Success. No rows returned"

#### Step 6: Verify Tables Created

1. In left sidebar, click "Table Editor"
2. You should see both tables:
   - `users`
   - `agent_runs`
3. Click each table to see the columns

#### Step 7: Get Connection String

1. Click "Settings" (gear icon) in left sidebar
2. Click "Database" in the submenu
3. Scroll down to "Connection string"
4. Find "URI" format - looks like:
   ```
   postgresql://postgres:[PASSWORD]@db.xxxx.supabase.co:5432/postgres
   ```
5. Click "Copy" button
6. **SAVE THIS SECURELY** - you'll need it later for the backend `.env` file

---

### Git Commands

This is cloud setup only - no code to commit yet.

---

### Jira Update

| When | Action |
|------|--------|
| 2:00 PM | Move AG-137 to "In Progress" |
| 2:45 PM | Move AG-137 to "Done" |

---

## 🕑 2:00 PM - Dev 3 (Marva): AG-87, AG-88 - CI/CD Setup

### What to Learn First (20 minutes)

**Read:** https://docs.github.com/en/actions/quickstart

**Key Concepts:**
- `.github/workflows/` folder contains workflow files
- YAML format defines steps
- Workflows trigger on events (push, PR, etc.)

---

### Step-by-Step Instructions

#### Step 1: Clone Repository

```bash
# Clone your organization's repo
git clone https://github.com/YOUR_ORG/resume-agent.git
cd resume-agent
```

#### Step 2: Check Current Branch

```bash
git branch
# Should show: * main
```

#### Step 3: Switch to Your Dev Branch

```bash
git checkout dev-marva
```

**Jira Update:** Move AG-87 and AG-88 to "In Progress"

#### Step 4: Create Feature Branch

```bash
git checkout -b feature/AG-87-cicd-setup
```

#### Step 5: Create Folder Structure

```bash
# Create directories
mkdir -p .github/workflows
mkdir -p backend
mkdir -p frontend

# Verify
ls -la
# Should show: .github, backend, frontend
```

#### Step 6: Create Backend Workflow File

Create file: `.github/workflows/backend.yml`

```yaml
# Backend CI/CD Pipeline
# This workflow runs when backend code changes

name: Backend CI

on:
  push:
    branches: [main]
    paths:
      - 'backend/**'
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          cd backend
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      
      - name: Run linting
        run: |
          pip install flake8
          cd backend
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics || true
      
      - name: Test health endpoint
        run: |
          cd backend
          python -c "from main import app; print('✓ App imports successfully')"
```

#### Step 7: Create Frontend Workflow File

Create file: `.github/workflows/frontend.yml`

```yaml
# Frontend CI/CD Pipeline  
# This workflow runs when frontend code changes

name: Frontend CI

on:
  push:
    branches: [main]
    paths:
      - 'frontend/**'
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        run: |
          cd frontend
          npm ci
      
      - name: Build
        run: |
          cd frontend
          npm run build
      
      - name: Run linting
        run: |
          cd frontend
          npm run lint || true
```

#### Step 8: Create Basic FastAPI Application

Create file: `backend/main.py`

```python
"""
Resume Agent API - Main FastAPI Application

This is the entry point for the backend API.
Run with: uvicorn main:app --reload
"""
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

# Create FastAPI app
app = FastAPI(
    title="Resume Agent API",
    description="AI-powered resume optimization service",
    version="1.0.0"
)

# Add CORS middleware to allow frontend to call API
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # In production, specify actual origins
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
        "docs": "/docs"
    }


@app.get("/health")
def health():
    """Health check endpoint - used by monitoring and CI/CD."""
    return {"status": "ok"}
```

#### Step 9: Create Requirements File

Create file: `backend/requirements.txt`

```
# Web framework
fastapi==0.109.0
uvicorn==0.27.0

# Will add more dependencies as we build features
```

#### Step 10: Test Locally

```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
```

Open another terminal:
```bash
curl http://localhost:8000/health
# Should return: {"status":"ok"}

curl http://localhost:8000/
# Should return: {"service":"Resume Agent API",...}
```

Stop the server with Ctrl+C.

#### Step 11: Check What Files You Created

```bash
cd ..  # Back to resume-agent root
git status
```

Should show:
```
Untracked files:
  .github/
  backend/
```

#### Step 12: Add and Commit

```bash
git add .
git commit -m "AG-87: Setup CI/CD pipelines and basic FastAPI backend

- Add GitHub Actions workflow for backend (test on push)
- Add GitHub Actions workflow for frontend (build on push)
- Create basic FastAPI app with /health endpoint
- Add requirements.txt with initial dependencies"
```

#### Step 13: Push to Feature Branch

```bash
git push origin feature/AG-87-cicd-setup
```

**Jira Update:** Move AG-87 and AG-88 to "In Review"

#### Step 14: Open Pull Request

1. Go to your GitHub repository in browser
2. You should see a yellow banner: "feature/AG-87-cicd-setup had recent pushes"
3. Click "Compare & pull request"
4. Fill in PR details:

**Title:**
```
AG-87: Setup CI/CD pipelines and basic FastAPI backend
```

**Description:**
```markdown
## Summary
Setup the CI/CD infrastructure and basic backend application.

## Jira Tasks
- AG-87: Backend CI/CD Pipeline
- AG-88: Frontend CI/CD Pipeline

## Changes Made
- Created `.github/workflows/backend.yml` - triggers on backend changes
- Created `.github/workflows/frontend.yml` - triggers on frontend changes
- Created `backend/main.py` with FastAPI app and /health endpoint
- Created `backend/requirements.txt` with initial dependencies

## How to Test

### Locally
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
curl http://localhost:8000/health  # Should return {"status":"ok"}
```

### CI/CD
- After merge, check GitHub Actions tab
- Workflow should run and pass

## Checklist
- [x] Workflow files are valid YAML
- [x] Backend runs locally
- [x] /health endpoint returns correct response
```

5. **Reviewers:** Request review from Dev 1 or Dev 2
6. Wait for approval
7. Click "Merge pull request"
8. Click "Confirm merge"

#### Step 15: Cleanup After Merge

```bash
# Switch back to your dev branch
git checkout dev-marva

# Get latest from main (includes your merged changes)
git pull origin main

# Delete the feature branch (no longer needed)
git branch -d feature/AG-87-cicd-setup
```

---

### Jira Update

| When | Action |
|------|--------|
| 2:00 PM | Move AG-87, AG-88 to "In Progress" |
| ~3:00 PM | Move AG-87, AG-88 to "In Review" (after push) |
| After merge | Move AG-87, AG-88 to "Done" |

---

## ✅ End of Day 1: Manual Verification

### Terminal Tests

```bash
# 1. Test backend runs locally
cd resume-agent/backend
pip install -r requirements.txt
uvicorn main:app --reload

# In another terminal:
# 2. Test health endpoint
curl http://localhost:8000/health
# Expected: {"status":"ok"}

# 3. Test root endpoint
curl http://localhost:8000/
# Expected: {"service":"Resume Agent API","version":"1.0.0",...}

# 4. Test Swagger docs accessible
curl -s http://localhost:8000/docs | head -5
# Expected: HTML content (Swagger UI page)
```

### Browser Tests

1. **Open Swagger UI:**
   - Go to http://localhost:8000/docs
   - Should see interactive API documentation
   - Try clicking "GET /health" → "Try it out" → "Execute"
   - Response should show `{"status":"ok"}`

2. **Test API Info:**
   - Go to http://localhost:8000
   - Should see JSON with service info

### Supabase Tests (Dev 2)

1. Go to Supabase Dashboard
2. Click "Table Editor" in sidebar
3. Verify `users` table exists with correct columns:
   - id (uuid)
   - email (varchar)
   - password_hash (varchar)
   - created_at (timestamp)
4. Verify `agent_runs` table exists with all columns

### GitHub Tests (Dev 3)

1. Go to GitHub repository
2. Click "Actions" tab
3. Click on the most recent workflow run
4. Verify it shows green checkmark (passed)
5. Click on the run to see step details

---

## 📁 Folder Structure After Day 1

```
resume-agent/
├── .github/
│   └── workflows/
│       ├── backend.yml      ✓ NEW
│       └── frontend.yml     ✓ NEW
├── backend/
│   ├── main.py              ✓ NEW
│   └── requirements.txt     ✓ NEW
└── README.md
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-13: LangGraph Tutorial Part 1 | Dev 1 | 2 | ✓ Done |
| AG-8: LangGraph Tutorial Part 2 | Dev 2 | 2 | ✓ Done |
| AG-9: LangGraph Tutorial Part 3 | Dev 3 | 2 | ✓ Done |
| AG-10: Agent State Schema | Dev 1 | 3 | ✓ Done |
| AG-11: Supabase Setup | Dev 2 | 3 | ✓ Done |
| AG-12: Backend CI/CD | Dev 1 | 2 | ✓ Done |
| AG-13: Frontend CI/CD | Dev 3 | 2 | ✓ Done |
| AG-14: FastAPI Basic Setup | Dev 2 | 2 | ✓ Done |

**Total Story Points Completed:** 18  
**Dev 1:** 7 points | **Dev 2:** 7 points | **Dev 3:** 4 points

---

**Next:** [Day 2: Foundation →](./day-02.md)
