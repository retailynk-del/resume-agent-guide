# Day 14: History Backend

> **Date:** Sprint 2, Day 14  
> **Focus:** Build run history backend infrastructure  
> **Vertical Slice:** Save and retrieve past optimizations from database

---

## 🎯 Today's Goal

By end of today:
- ✅ Database indexes optimized for history queries (Dev 1)
- ✅ Agent runs fully persisted to database (Dev 2)
- ✅ History API endpoints created (Dev 3)
- ✅ User can retrieve list of past optimization runs

---

## 📋 Jira Tasks

### Task AG-54: History Database Schema Enhancement

| Field | Value |
|-------|-------|
| **Task ID** | AG-54 |
| **Title** | History Database Schema Enhancement |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Enhance agent_runs table with proper indexes and constraints to support efficient history queries. Ensure schema supports listing runs by user with pagination.

**Acceptance Criteria:**
- [ ] Add composite index on (user_id, created_at DESC)
- [ ] Verify all timestamp fields exist
- [ ] Add database performance tests
- [ ] Migration script documented
- [ ] Supabase dashboard verified
- [ ] PR approved and merged

---

### Task AG-55: Persist Full Run Data

| Field | Value |
|-------|-------|
| **Task ID** | AG-55 |
| **Title** | Persist Full Run Data |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |
| **Dependencies** | AG-54 |

**Description:**
Ensure agent optimization endpoint saves complete run data to database including all intermediate results, scores, and metadata for history display.

**Acceptance Criteria:**
- [ ] Verify all fields saved (job_description, resumes, scores, analysis)
- [ ] Cover letter saved if generated
- [ ] Timestamps set correctly
- [ ] Status field updated properly
- [ ] Error handling for database failures
- [ ] Tests verify data persistence
- [ ] PR approved and merged

---

### Task AG-56: History API Endpoints

| Field | Value |
|-------|-------|
| **Task ID** | AG-56 |
| **Title** | History API Endpoints |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 4 |
| **Sprint** | Sprint 2 |
| **Priority** | High |
| **Dependencies** | AG-55 |

**Description:**
Create API endpoints to retrieve user's optimization history. Support listing recent runs with pagination and fetching individual run details.

**Acceptance Criteria:**
- [ ] GET /api/agent/runs - list user's runs (paginated)
- [ ] GET /api/agent/runs/{id} - get single run details
- [ ] Requires authentication
- [ ] Returns runs sorted by date (newest first)
- [ ] Pagination with limit/offset parameters
- [ ] API documentation updated
- [ ] Tests cover success and error cases
- [ ] PR approved and merged

---

## 🕘 9:00 AM - Daily Standup

**Shabas:** "I'll add database indexes for fast history queries today."

**Sinan:** "I'll verify the agent endpoint is saving all the data we need for the history page."

**Marva:** "I'll create the history API endpoints with pagination support."

---

## 👨‍💻 Dev 1 (Shabas): AG-54 - History Database Schema Enhancement

### Step 1: Create Branch

```bash
git checkout shabas-Dev && git pull origin main
git checkout -b feature/AG-54-history-schema
```

**Jira:** Move AG-54 to "In Progress"

### Step 2: Review Current Schema

Check existing `backend/models/agent_run.py`:

```python
# Current AgentRun model has:
# - id, user_id (with FK to users)
# - job_description, original_resume, modified_resume
# - cover_letter, cover_letter_word_count (from Day 13)
# - ats_score_before, ats_score_after, improvement_delta
# - job_requirements, resume_analysis, improvement_plan (JSONB)
# - iteration_count, status
# - created_at, completed_at
```

**Schema looks good!** Just need to verify indexes.

### Step 3: Create Index Migration

Create `backend/migrations/add_history_indexes.sql`:

```sql
-- Add index for efficient history queries
-- Composite index on (user_id, created_at DESC) for sorting

CREATE INDEX IF NOT EXISTS idx_agent_runs_user_created 
ON agent_runs (user_id, created_at DESC);

-- Index on status for filtering
CREATE INDEX IF NOT EXISTS idx_agent_runs_status 
ON agent_runs (status);

-- Analyze the table for query planner
ANALYZE agent_runs;
```

### Step 4: Apply Migration in Supabase

**Steps:**
1. Go to https://supabase.com → Your Project → SQL Editor
2. Create new query
3. Paste the SQL above
4. Click "Run"
5. Verify: Go to Database → agent_runs → Indexes tab

**Expected indexes:**
- `agent_runs_pkey` (PRIMARY KEY on id)
- `idx_agent_runs_user_id` (existing from Day 8)
- `idx_agent_runs_user_created` (NEW - composite)
- `idx_agent_runs_status` (NEW)

### Step 5: Test Index Performance

Create `backend/test_history_performance.py`:

```python
"""
Test history query performance with indexes.
"""
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker
import os
from dotenv import load_dotenv
import time

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)


def test_history_query_performance():
    """Test query performance for history page."""
    
    db = SessionLocal()
    
    # Simulate history query (what we'll use in API)
    query = text("""
        SELECT id, job_description, ats_score_before, ats_score_after, 
               improvement_delta, created_at, cover_letter IS NOT NULL as has_cover_letter
        FROM agent_runs
        WHERE user_id = :user_id
        ORDER BY created_at DESC
        LIMIT 20
    """)
    
    # Test with a sample user_id (use a real one from your database)
    # For testing, you can use any UUID
    test_user_id = "00000000-0000-0000-0000-000000000000"
    
    # Measure query time
    start = time.time()
    result = db.execute(query, {"user_id": test_user_id})
    rows = result.fetchall()
    end = time.time()
    
    query_time = (end - start) * 1000  # Convert to milliseconds
    
    print(f"✓ History query completed in {query_time:.2f}ms")
    print(f"✓ Found {len(rows)} runs")
    
    # Query should be fast even with many runs
    assert query_time < 100, f"Query too slow: {query_time}ms"
    
    db.close()
    
    print("\n✓ Performance test passed!")


if __name__ == "__main__":
    test_history_query_performance()
```

Run test:

```bash
cd backend
python test_history_performance.py
```

### Step 6: Document Schema

Create `docs/database-schema.md`:

```markdown
# Database Schema

## Tables

### users
- `id` UUID PRIMARY KEY
- `email` VARCHAR(255) UNIQUE NOT NULL
- `password_hash` TEXT NOT NULL
- `created_at` TIMESTAMP DEFAULT NOW()

**Indexes:**
- PRIMARY KEY on id
- UNIQUE on email

---

### agent_runs
- `id` UUID PRIMARY KEY
- `user_id` UUID REFERENCES users(id)
- `job_description` TEXT NOT NULL
- `original_resume` TEXT NOT NULL
- `modified_resume` TEXT
- `cover_letter` TEXT
- `cover_letter_word_count` INTEGER
- `ats_score_before` FLOAT
- `ats_score_after` FLOAT
- `improvement_delta` FLOAT
- `job_requirements` JSONB
- `resume_analysis` JSONB
- `improvement_plan` JSONB
- `iteration_count` INTEGER DEFAULT 0
- `status` VARCHAR(20) DEFAULT 'pending'
- `created_at` TIMESTAMP DEFAULT NOW()
- `completed_at` TIMESTAMP

**Indexes:**
- PRIMARY KEY on id
- INDEX on user_id
- COMPOSITE INDEX on (user_id, created_at DESC) - for history queries
- INDEX on status - for filtering

**Foreign Keys:**
- user_id → users(id)

---

## Common Queries

### Get user's recent optimizations
```sql
SELECT * FROM agent_runs
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20;
```
Uses index: `idx_agent_runs_user_created`

### Get single run with all details
```sql
SELECT * FROM agent_runs
WHERE id = $1 AND user_id = $2;
```
Uses index: PRIMARY KEY

### Count user's total runs
```sql
SELECT COUNT(*) FROM agent_runs
WHERE user_id = $1;
```
Uses index: `idx_agent_runs_user_id`
```

### Step 7: Commit Schema Changes

```bash
git add backend/migrations/add_history_indexes.sql backend/test_history_performance.py docs/database-schema.md
git commit -m "AG-54: Enhance database schema for history queries

- Add composite index on (user_id, created_at DESC)
- Add index on status field
- Performance test for history queries
- Complete schema documentation
- Query execution < 100ms verified"

git push origin feature/AG-54-history-schema
```

**Jira:** Move AG-54 to "Code Review"

---

## 👨‍💻 Dev 2 (Sinan): AG-55 - Persist Full Run Data

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-55-persist-runs
```

**Jira:** Move AG-55 to "In Progress"

### Step 2: Review Agent Endpoint

Check `backend/routers/agent.py` - the `/optimize` endpoint:

```python
# Current implementation from Day 13:
# - Already creates AgentRun record
# - Saves all inputs (job_description, original_resume)
# - Saves outputs (modified_resume, cover_letter)
# - Saves scores (before, after, delta)
# - Saves analysis (job_requirements, resume_analysis, improvement_plan)
# - Sets timestamps
```

**Looks complete!** Just need to verify and add error handling.

### Step 3: Add Error Handling

Edit `backend/routers/agent.py` - enhance error handling:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime
import traceback

from models.agent_run import AgentRun
from database.connection import get_db
from agent.workflow import run_optimization
from auth.dependencies import get_current_user_id

router = APIRouter(prefix="/api/agent", tags=["agent"])


class OptimizationRequest(BaseModel):
    resume_text: str = Field(..., min_length=100)
    job_description: str = Field(..., min_length=50)
    generate_cover_letter: bool = False


class OptimizationResponse(BaseModel):
    id: str
    original_resume: str
    modified_resume: str
    cover_letter: Optional[str] = None
    cover_letter_word_count: Optional[int] = None
    ats_score_before: float
    ats_score_after: float
    improvement_delta: float
    job_requirements: dict
    resume_analysis: dict
    improvement_plan: dict
    iteration_count: int
    status: str
    created_at: str
    completed_at: Optional[str] = None


@router.post("/optimize", response_model=OptimizationResponse)
def optimize_resume(
    request: OptimizationRequest,
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """
    Optimize resume for job description.
    Optionally generate cover letter.
    Saves complete run data to database for history.
    """
    
    # Create pending run record immediately
    run = AgentRun(
        user_id=user_id,
        job_description=request.job_description,
        original_resume=request.resume_text,
        status="running"
    )
    
    try:
        db.add(run)
        db.commit()
        db.refresh(run)
        
        # Run agent workflow
        result = run_optimization(
            job_description=request.job_description,
            resume=request.resume_text,
            user_id=user_id,
            run_id=str(run.id),
            generate_cover_letter=request.generate_cover_letter
        )
        
        # Update run with results
        run.modified_resume = result["modified_resume"]
        run.cover_letter = result.get("cover_letter")
        run.cover_letter_word_count = result.get("cover_letter_word_count")
        run.ats_score_before = result["ats_score_before"]
        run.ats_score_after = result["ats_score_after"]
        run.improvement_delta = result["improvement_delta"]
        run.job_requirements = result["job_requirements"]
        run.resume_analysis = result["resume_analysis"]
        run.improvement_plan = result["improvement_plan"]
        run.iteration_count = result.get("iteration_count", 0)
        run.status = "completed"
        run.completed_at = datetime.utcnow()
        
        db.commit()
        db.refresh(run)
        
        return OptimizationResponse(
            id=str(run.id),
            original_resume=run.original_resume,
            modified_resume=run.modified_resume,
            cover_letter=run.cover_letter,
            cover_letter_word_count=run.cover_letter_word_count,
            ats_score_before=run.ats_score_before,
            ats_score_after=run.ats_score_after,
            improvement_delta=run.improvement_delta,
            job_requirements=run.job_requirements,
            resume_analysis=run.resume_analysis,
            improvement_plan=run.improvement_plan,
            iteration_count=run.iteration_count,
            status=run.status,
            created_at=run.created_at.isoformat(),
            completed_at=run.completed_at.isoformat() if run.completed_at else None
        )
        
    except Exception as e:
        # Mark run as failed
        run.status = "failed"
        run.completed_at = datetime.utcnow()
        db.commit()
        
        # Log error
        print(f"ERROR in optimization: {str(e)}")
        print(traceback.format_exc())
        
        # Return error to user
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Optimization failed: {str(e)}"
        )
```

### Step 4: Test Data Persistence

Create `backend/test_data_persistence.py`:

```python
"""
Test that optimization runs are fully persisted.
"""
import requests
import os
from dotenv import load_dotenv

load_dotenv()

BASE_URL = "http://localhost:8000"


def test_full_data_persistence():
    """Test that all run data is saved to database."""
    
    # 1. Login
    login_response = requests.post(
        f"{BASE_URL}/api/auth/login",
        json={
            "email": "test@test.com",
            "password": "Test1234"
        }
    )
    
    assert login_response.status_code == 200
    token = login_response.json()["access_token"]
    
    headers = {"Authorization": f"Bearer {token}"}
    
    # 2. Run optimization WITH cover letter
    optimize_response = requests.post(
        f"{BASE_URL}/api/agent/optimize",
        headers=headers,
        json={
            "resume_text": "John Smith. Senior Engineer with 6 years Python and React experience." * 5,
            "job_description": "Senior Software Engineer. 5+ years Python, React required." * 3,
            "generate_cover_letter": True
        }
    )
    
    assert optimize_response.status_code == 200
    result = optimize_response.json()
    run_id = result["id"]
    
    # 3. Verify all fields are present
    required_fields = [
        "id", "original_resume", "modified_resume",
        "ats_score_before", "ats_score_after", "improvement_delta",
        "job_requirements", "resume_analysis", "improvement_plan",
        "created_at", "completed_at", "status", "iteration_count"
    ]
    
    for field in required_fields:
        assert field in result, f"Missing field: {field}"
        assert result[field] is not None, f"Field is None: {field}"
    
    # 4. Verify cover letter was saved
    assert result["cover_letter"] is not None
    assert result["cover_letter_word_count"] > 0
    assert result["status"] == "completed"
    
    # 5. Fetch the run again to verify database persistence
    get_response = requests.get(
        f"{BASE_URL}/api/agent/runs/{run_id}",
        headers=headers
    )
    
    assert get_response.status_code == 200
    fetched_run = get_response.json()
    
    # Verify data matches
    assert fetched_run["id"] == run_id
    assert fetched_run["cover_letter"] == result["cover_letter"]
    assert fetched_run["ats_score_after"] == result["ats_score_after"]
    
    print("✓ All data persisted correctly to database!")
    print(f"✓ Run ID: {run_id}")
    print(f"✓ Score improvement: +{result['improvement_delta']:.1f} points")
    print(f"✓ Cover letter: {result['cover_letter_word_count']} words")


if __name__ == "__main__":
    print("Testing data persistence...\n")
    test_full_data_persistence()
    print("\n✓ All persistence tests passed!")
```

Run test:

```bash
# Start backend first
cd backend
uvicorn main:app --reload

# In another terminal:
cd backend
python test_data_persistence.py
```

### Step 5: Commit Persistence Enhancements

```bash
git add backend/routers/agent.py backend/test_data_persistence.py
git commit -m "AG-55: Ensure full run data persistence

- Create run record immediately with 'running' status
- Update all fields after workflow completion
- Handle errors with 'failed' status
- Set completed_at timestamp
- Comprehensive persistence test
- All fields verified stored in database"

git push origin feature/AG-55-persist-runs
```

**Jira:** Move AG-55 to "Code Review"

---

## 👩‍💻 Dev 3 (Marva): AG-56 - History API Endpoints

### Step 1: Create Branch

```bash
git checkout marva-Dev && git pull origin main
git checkout -b feature/AG-56-history-api
```

**Jira:** Move AG-56 to "In Progress"

### Step 2: Update Agent Router

Edit `backend/routers/agent.py` - add history endpoints:

```python
# Add to existing router

from typing import List


class RunListItem(BaseModel):
    """Summary for history list."""
    id: str
    job_description: str  # First 100 chars
    ats_score_before: float
    ats_score_after: float
    improvement_delta: float
    has_cover_letter: bool
    status: str
    created_at: str


@router.get("/runs", response_model=List[RunListItem])
def list_optimization_runs(
    limit: int = 20,
    offset: int = 0,
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """
    Get user's optimization history.
    
    Query params:
    - limit: Max results (default 20, max 100)
    - offset: Skip N results (for pagination)
    
    Returns list sorted by created_at DESC (newest first).
    """
    
    # Validate pagination parameters
    if limit < 1 or limit > 100:
        raise HTTPException(
            status_code=400,
            detail="Limit must be between 1 and 100"
        )
    
    if offset < 0:
        raise HTTPException(
            status_code=400,
            detail="Offset must be >= 0"
        )
    
    # Query database
    runs = db.query(AgentRun).filter(
        AgentRun.user_id == user_id
    ).order_by(
        AgentRun.created_at.desc()
    ).limit(limit).offset(offset).all()
    
    # Format response
    result = []
    for run in runs:
        # Truncate job description for list view
        job_desc_preview = run.job_description[:100]
        if len(run.job_description) > 100:
            job_desc_preview += "..."
        
        result.append(RunListItem(
            id=str(run.id),
            job_description=job_desc_preview,
            ats_score_before=run.ats_score_before or 0.0,
            ats_score_after=run.ats_score_after or 0.0,
            improvement_delta=run.improvement_delta or 0.0,
            has_cover_letter=run.cover_letter is not None,
            status=run.status,
            created_at=run.created_at.isoformat() if run.created_at else ""
        ))
    
    return result


@router.get("/runs/{run_id}", response_model=OptimizationResponse)
def get_optimization_run(
    run_id: str,
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """
    Get full details of a specific optimization run.
    
    Returns complete run data including all analysis and generated content.
    """
    
    # Query run
    run = db.query(AgentRun).filter(
        AgentRun.id == run_id,
        AgentRun.user_id == user_id
    ).first()
    
    if not run:
        raise HTTPException(
            status_code=404,
            detail="Optimization run not found"
        )
    
    return OptimizationResponse(
        id=str(run.id),
        original_resume=run.original_resume,
        modified_resume=run.modified_resume or "",
        cover_letter=run.cover_letter,
        cover_letter_word_count=run.cover_letter_word_count,
        ats_score_before=run.ats_score_before or 0.0,
        ats_score_after=run.ats_score_after or 0.0,
        improvement_delta=run.improvement_delta or 0.0,
        job_requirements=run.job_requirements or {},
        resume_analysis=run.resume_analysis or {},
        improvement_plan=run.improvement_plan or {},
        iteration_count=run.iteration_count or 0,
        status=run.status,
        created_at=run.created_at.isoformat() if run.created_at else "",
        completed_at=run.completed_at.isoformat() if run.completed_at else None
    )


@router.get("/runs/count")
def get_runs_count(
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """Get total number of optimization runs for user."""
    
    count = db.query(AgentRun).filter(
        AgentRun.user_id == user_id
    ).count()
    
    return {"total": count}
```

### Step 3: Test History API

Create `backend/test_history_api.py`:

```python
"""
Test history API endpoints.
"""
import requests
import os
from dotenv import load_dotenv

load_dotenv()

BASE_URL = "http://localhost:8000"


def get_auth_token():
    """Helper to get auth token."""
    response = requests.post(
        f"{BASE_URL}/api/auth/login",
        json={"email": "test@test.com", "password": "Test1234"}
    )
    return response.json()["access_token"]


def test_list_runs():
    """Test listing optimization runs."""
    token = get_auth_token()
    headers = {"Authorization": f"Bearer {token}"}
    
    # 1. Get runs list
    response = requests.get(
        f"{BASE_URL}/api/agent/runs",
        headers=headers
    )
    
    assert response.status_code == 200
    runs = response.json()
    
    assert isinstance(runs, list)
    
    if len(runs) > 0:
        # Verify structure
        run = runs[0]
        required_fields = [
            "id", "job_description", "ats_score_before",
            "ats_score_after", "improvement_delta",
            "has_cover_letter", "status", "created_at"
        ]
        
        for field in required_fields:
            assert field in run, f"Missing field: {field}"
        
        # Verify sorted by date (newest first)
        if len(runs) > 1:
            date1 = runs[0]["created_at"]
            date2 = runs[1]["created_at"]
            assert date1 >= date2, "Not sorted correctly"
    
    print(f"✓ Found {len(runs)} runs in history")


def test_pagination():
    """Test pagination parameters."""
    token = get_auth_token()
    headers = {"Authorization": f"Bearer {token}"}
    
    # Test with limit
    response = requests.get(
        f"{BASE_URL}/api/agent/runs?limit=5",
        headers=headers
    )
    
    assert response.status_code == 200
    runs = response.json()
    assert len(runs) <= 5
    
    print(f"✓ Pagination works (limit=5, got {len(runs)} runs)")


def test_get_single_run():
    """Test fetching single run details."""
    token = get_auth_token()
    headers = {"Authorization": f"Bearer {token}"}
    
    # Get list first
    list_response = requests.get(
        f"{BASE_URL}/api/agent/runs?limit=1",
        headers=headers
    )
    
    runs = list_response.json()
    
    if len(runs) == 0:
        print("⚠ No runs to test with")
        return
    
    run_id = runs[0]["id"]
    
    # Get full details
    detail_response = requests.get(
        f"{BASE_URL}/api/agent/runs/{run_id}",
        headers=headers
    )
    
    assert detail_response.status_code == 200
    run = detail_response.json()
    
    # Verify has full data
    assert run["id"] == run_id
    assert run["original_resume"] is not None
    assert run["modified_resume"] is not None
    assert "job_requirements" in run
    
    print(f"✓ Fetched full details for run {run_id[:8]}...")


def test_get_count():
    """Test runs count endpoint."""
    token = get_auth_token()
    headers = {"Authorization": f"Bearer {token}"}
    
    response = requests.get(
        f"{BASE_URL}/api/agent/runs/count",
        headers=headers
    )
    
    assert response.status_code == 200
    data = response.json()
    assert "total" in data
    assert isinstance(data["total"], int)
    
    print(f"✓ User has {data['total']} total runs")


def test_not_found():
    """Test 404 for non-existent run."""
    token = get_auth_token()
    headers = {"Authorization": f"Bearer {token}"}
    
    fake_id = "00000000-0000-0000-0000-000000000000"
    response = requests.get(
        f"{BASE_URL}/api/agent/runs/{fake_id}",
        headers=headers
    )
    
    assert response.status_code == 404
    
    print("✓ Returns 404 for non-existent run")


if __name__ == "__main__":
    print("Testing History API...\n")
    
    test_list_runs()
    test_pagination()
    test_get_single_run()
    test_get_count()
    test_not_found()
    
    print("\n✓ All history API tests passed!")
```

Run tests:

```bash
# Backend should be running
cd backend
python test_history_api.py
```

### Step 4: Commit History API

```bash
git add backend/routers/agent.py backend/test_history_api.py
git commit -m "AG-56: Create history API endpoints

- GET /api/agent/runs - list runs with pagination
- GET /api/agent/runs/{id} - get full run details
- GET /api/agent/runs/count - get total count
- Pagination with limit/offset
- Sorted by created_at DESC
- Truncate job_description in list view
- Comprehensive API tests
- 404 handling for missing runs"

git push origin feature/AG-56-history-api
```

**Jira:** Move AG-56 to "Code Review"

---

## 📞 2:00 PM - Review Calls

### Shabas → Sinan: Review AG-54

**Shabas:** "Sinan, I've added database indexes for history queries. Can you review?"

**Sinan:** *Reviews migration SQL and performance tests*

"Perfect! The composite index on (user_id, created_at DESC) will make history page queries very fast. Performance test confirms < 100ms. Approved! ✅"

**Jira:** AG-54 moved to "Done"

---

### Sinan → Marva: Review AG-55

**Sinan:** "Marva, I've enhanced the persistence logic with better error handling. Please review!"

**Marva:** *Reviews agent.py changes and persistence tests*

"Excellent work! The status tracking (running → completed/failed) is clean, and the error handling properly marks failed runs. Tests confirm all data is saved. Approved! ✅"

**Jira:** AG-55 moved to "Done"

---

### Marva → Shabas: Review AG-56

**Marva:** "Shabas, I've created the history API endpoints with pagination. Can you review?"

**Shabas:** *Reviews new endpoints and API tests*

"Great implementation! The pagination works smoothly, job description truncation is smart for list view, and the tests cover all cases. Approved! ✅"

**Jira:** AG-56 moved to "Done"

---

## 🔀 3:00 PM - Merge to Main

```bash
# Merge AG-54
git checkout main
git pull origin main
git merge feature/AG-54-history-schema
git push origin main

# Merge AG-55
git merge feature/AG-55-persist-runs
git push origin main

# Merge AG-56
git merge feature/AG-56-history-api
git push origin main

# Clean up
git branch -d feature/AG-54-history-schema
git branch -d feature/AG-55-persist-runs
git branch -d feature/AG-56-history-api
```

---

## ✅ 3:30 PM - End-to-End Verification

### Test Complete History Backend

```bash
# 1. Login
TOKEN=$(curl -s -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"Test1234"}' | jq -r '.access_token')

# 2. Run 2-3 optimizations to build history
for i in {1..3}; do
  curl -X POST http://localhost:8000/api/agent/optimize \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
      "resume_text": "Developer with '"$i"' years experience in Python and React...",
      "job_description": "Looking for senior developer with Python skills...",
      "generate_cover_letter": false
    }' | jq '.id'
  
  echo "Optimization $i completed"
  sleep 2
done

# 3. Get history list
curl -X GET "http://localhost:8000/api/agent/runs?limit=10" \
  -H "Authorization: Bearer $TOKEN" | jq '.'

# 4. Get total count
curl -X GET http://localhost:8000/api/agent/runs/count \
  -H "Authorization: Bearer $TOKEN" | jq '.'

# 5. Get details of first run
RUN_ID=$(curl -s -X GET "http://localhost:8000/api/agent/runs?limit=1" \
  -H "Authorization: Bearer $TOKEN" | jq -r '.[0].id')

curl -X GET "http://localhost:8000/api/agent/runs/$RUN_ID" \
  -H "Authorization: Bearer $TOKEN" | jq '.ats_score_after'
```

### Verification Checklist

| # | Test | Expected Result | ✓ |
|---|------|-----------------|---|
| 1 | Database indexes exist | Composite index on (user_id, created_at DESC) | |
| 2 | Run data persists | All fields saved to database | |
| 3 | GET /api/agent/runs | Returns list sorted by date | |
| 4 | Pagination works | limit=5 returns max 5 runs | |
| 5 | GET /api/agent/runs/{id} | Returns full run details | |
| 6 | GET /api/agent/runs/count | Returns total count | |
| 7 | Job description truncated | List view shows first 100 chars | |
| 8 | Cover letter flag | has_cover_letter boolean correct | |
| 9 | Authentication required | 401 without token | |
| 10 | 404 for missing run | Invalid ID returns 404 | |

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-54: History Database Schema | Dev 1 (Shabas) | 3 | ✅ Done |
| AG-55: Persist Full Run Data | Dev 2 (Sinan) | 3 | ✅ Done |
| AG-56: History API Endpoints | Dev 3 (Marva) | 4 | ✅ Done |

**Total Story Points Completed:** 10  
**Dev 1 (Shabas):** 3 points | **Dev 2 (Sinan):** 3 points | **Dev 3 (Marva):** 4 points

---

## 🎉 History Backend Complete!

Day 14 achievements:
- ✅ Database indexes optimized for fast queries
- ✅ Complete run data persistence with error handling
- ✅ History API with pagination support
- ✅ List view and detail view endpoints
- ✅ All tests passing

**Backend ready for History UI on Day 15! 📜**

---

**← [Day 13](./day-13.md)** | **[Day 15: History UI →](./day-15.md)**
Write the cover letter now:"""


def generate_cover_letter(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Generate personalized cover letter.
    
    Should be called after job requirements extraction and resume analysis.
    """
    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0.7,  # Some creativity is good for cover letters
        api_key=os.getenv("OPENAI_API_KEY")
    )
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """You are an expert cover letter writer. You create 
        compelling, personalized cover letters that highlight the candidate's 
        relevant experience and enthusiasm for the role."""),
        ("user", COVER_LETTER_PROMPT)
    ])
    
    chain = prompt | llm
    
    # Extract key skills to highlight
    requirements = state.get("job_requirements", {})
    analysis = state.get("resume_analysis", {})
    
    must_have = requirements.get("must_have_skills", [])
    current = analysis.get("current_skills", [])
    
    # Find matching skills to highlight
    matching_skills = [s for s in must_have if any(
        s.lower() in c.lower() or c.lower() in s.lower() 
        for c in current
    )]
    
    result = chain.invoke({
        "job_requirements": json.dumps(requirements),
        "resume_analysis": json.dumps(analysis),
        "key_skills": ", ".join(matching_skills[:5]) or "relevant experience"
    })
    
    cover_letter = result.content
    
    decision = {
        "node": "generate_cover_letter",
        "action": "generated_cover_letter",
        "length": len(cover_letter),
        "skills_highlighted": len(matching_skills)
    }
    
    return {
        "cover_letter": cover_letter,
        "decision_log": state.get("decision_log", []) + [decision]
    }
```

### Step 3: Update Workflow

Update file: `backend/agent/workflow.py`:

```python
# Add import
from .nodes.cover_letter import generate_cover_letter

# In create_agent_workflow():
# Add node
workflow.add_node("generate_cover_letter", generate_cover_letter)

# Add edge after analyze_resume (parallel path)
# Or add after log_decision before END
workflow.add_edge("log_decision", "generate_cover_letter")
workflow.add_edge("generate_cover_letter", END)

# Update conditional edges
workflow.add_conditional_edges(
    "log_decision",
    should_iterate,
    {
        "iterate": "plan_improvements",
        "finish": "generate_cover_letter"  # Changed from END
    }
)
```

### Step 4: Test

```bash
cd backend/agent
python test_workflow.py

# Check result includes cover_letter
```

### Step 5: Commit and PR

```bash
git add backend/agent/
git commit -m "AG-23: Add cover letter generation node

- Create LLM-based cover letter generation
- Integrate into workflow after final score
- Highlight matching skills"

git push origin feature/AG-23-cover-letter
```

---

## ✅ Day 14: Verification

- [ ] Cover letter appears in workflow result
- [ ] Letter is personalized to job/resume
- [ ] Length is reasonable (~400 words)

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-54: History Database Schema | Dev 1 | 3 | ✓ Done |
| AG-55: Save Runs to Database | Dev 2 | 3 | ✓ Done |
| AG-56: History API Endpoints | Dev 3 | 4 | ✓ Done |

**Total Story Points Completed:** 10  
**Dev 1:** 3 points | **Dev 2:** 3 points | **Dev 3:** 4 points

---

**← [Day 13](./day-13.md)** | **[Day 15: History UI →](./day-15.md)**
