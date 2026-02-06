# Day 12: Iterative Agent

> **Date:** Sprint 2, Day 12  
> **Focus:** Add iteration capability to agent workflow  
> **Vertical Slice:** Agent loops to improve score

---

## 🎯 Today's Goal

By end of today:
- ✅ Decision node with loop logic created (Dev 2 - more LangGraph!)
- ✅ Conditional edge implementation (Dev 1)
- ✅ Iteration testing complete (Dev 3)
- ✅ Agent loops up to 3x until target score reached

---

## 📋 Jira Tasks

### Task AG-48: Decision Node (Loop Logic)

| Field | Value |
|-------|-------|
| **Task ID** | AG-48 |
| **Title** | Decision Node (Loop Logic) |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Create LangGraph decision node that determines whether to iterate or finish based on score and iteration count. Dev 2 gets more advanced LangGraph experience!

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/decision.py`
- [ ] Checks if score >= target (70) OR max iterations (3) reached
- [ ] Returns "continue" or "finish"
- [ ] Updates iteration_count in state
- [ ] Logs decision reasoning
- [ ] PR merged

---

### Task AG-49: Conditional Edge Implementation

| Field | Value |
|-------|-------|
| **Task ID** | AG-49 |
| **Title** | Conditional Edge Implementation |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Update workflow.py to use conditional edges that route based on decision node output. Creates loop back to planning for iteration.

**Acceptance Criteria:**
- [ ] Add decision node to workflow
- [ ] Conditional edge after rescore: decision node
- [ ] If "continue" → route back to planning node
- [ ] If "finish" → route to END
- [ ] State properly maintained through loop
- [ ] PR merged

---

### Task AG-50: Iteration Testing

| Field | Value |
|-------|-------|
| **Task ID** | AG-50 |
| **Title** | Iteration Testing |
| **Type** | Task |
| **Epic** | Testing & QA |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Test the iterative workflow with various scenarios: early stopping, max iterations, and score improvement patterns.

**Acceptance Criteria:**
- [ ] Test: Agent stops when score >= 70
- [ ] Test: Agent stops at max 3 iterations
- [ ] Test: Score improves each iteration
- [ ] Test: iteration_count increments correctly
- [ ] Test: Decision log tracks all iterations
- [ ] All tests pass
- [ ] PR merged

---

## 👨‍💻 Dev 2: AG-42 - Save Runs

### Update Agent Router

Update `backend/api/agent.py`:

```python
"""Agent API with database persistence."""
import uuid
from datetime import datetime

from fastapi import APIRouter, HTTPException, Depends, status
from pydantic import BaseModel
from sqlalchemy.orm import Session

from api.auth import get_current_user_id
from database.connection import get_db
from database.models.run import AgentRun
from agent.workflow import run_optimization

router = APIRouter(prefix="/api/agent", tags=["Agent"])


class OptimizationRequest(BaseModel):
    job_description: str
    resume: str


class OptimizationResponse(BaseModel):
    run_id: str
    status: str
    ats_score_before: float = None
    ats_score_after: float = None
    improvement_delta: float = None
    modified_resume: str = None
    improvement_plan: dict = None
    job_requirements: dict = None


@router.post("/run", response_model=OptimizationResponse)
def run_agent(
    request: OptimizationRequest,
    user_id: str = Depends(get_current_user_id),
    db: Session = Depends(get_db)
):
    """Run optimization and save to database."""
    if len(request.job_description.strip()) < 50:
        raise HTTPException(400, "Job description too short")
    
    if len(request.resume.strip()) < 100:
        raise HTTPException(400, "Resume too short")
    
    run_id = str(uuid.uuid4())
    
    # Create initial run record
    db_run = AgentRun(
        id=run_id,
        user_id=user_id,
        job_description=request.job_description,
        original_resume=request.resume,
        status="running"
    )
    db.add(db_run)
    db.commit()
    
    try:
        # Run optimization
        result = run_optimization(
            job_description=request.job_description,
            resume=request.resume,
            user_id=user_id,
            run_id=run_id
        )
        
        # Update run record
        db_run.modified_resume = result.get("modified_resume")
        db_run.ats_score_before = result.get("ats_score_before")
        db_run.ats_score_after = result.get("ats_score_after")
        db_run.improvement_delta = result.get("improvement_delta")
        db_run.job_requirements = result.get("job_requirements")
        db_run.improvement_plan = result.get("improvement_plan")
        db_run.status = "completed"
        db_run.completed_at = datetime.utcnow()
        
        db.commit()
        
        return OptimizationResponse(
            run_id=run_id,
            status="completed",
            ats_score_before=result.get("ats_score_before"),
            ats_score_after=result.get("ats_score_after"),
            improvement_delta=result.get("improvement_delta"),
            modified_resume=result.get("modified_resume"),
            improvement_plan=result.get("improvement_plan"),
            job_requirements=result.get("job_requirements")
        )
        
    except Exception as e:
        db_run.status = "failed"
        db.commit()
        raise HTTPException(500, f"Optimization failed: {str(e)}")
```

---

## 👨‍💻 Dev 1: AG-43 - History Endpoint

### Add to Agent Router

Add to `backend/api/agent.py`:

```python
from typing import List
from pydantic import BaseModel


class RunSummary(BaseModel):
    id: str
    ats_score_before: float = None
    ats_score_after: float = None
    improvement_delta: float = None
    status: str
    created_at: str


@router.get("/history", response_model=List[RunSummary])
def get_run_history(
    user_id: str = Depends(get_current_user_id),
    db: Session = Depends(get_db),
    limit: int = 10
):
    """Get user's run history."""
    runs = db.query(AgentRun)\
        .filter(AgentRun.user_id == user_id)\
        .order_by(AgentRun.created_at.desc())\
        .limit(limit)\
        .all()
    
    return [
        RunSummary(
            id=str(run.id),
            ats_score_before=run.ats_score_before,
            ats_score_after=run.ats_score_after,
            improvement_delta=run.improvement_delta,
            status=run.status,
            created_at=run.created_at.isoformat() if run.created_at else None
        )
        for run in runs
    ]


@router.get("/run/{run_id}")
def get_run_detail(
    run_id: str,
    user_id: str = Depends(get_current_user_id),
    db: Session = Depends(get_db)
):
    """Get full details of a specific run."""
    run = db.query(AgentRun)\
        .filter(AgentRun.id == run_id, AgentRun.user_id == user_id)\
        .first()
    
    if not run:
        raise HTTPException(404, "Run not found")
    
    return run.to_full_dict()
```

---

## ✅ Day 12: Manual Verification

```bash
# 1. Login and get token
TOKEN=$(curl -s -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"Test1234"}' | jq -r '.access_token')

# 2. Run optimization
curl -X POST http://localhost:8000/api/agent/run \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"job_description":"Need Python dev with Django...(50+ chars)","resume":"John Doe developer...(100+ chars)"}'

# 3. Check history
curl http://localhost:8000/api/agent/history \
  -H "Authorization: Bearer $TOKEN"

# 4. Check Supabase - should see run in agent_runs table
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-48: Decision Node (Loop Logic) | Dev 2 | 5 | ✓ Done |
| AG-49: Conditional Edge Implementation | Dev 1 | 3 | ✓ Done |
| AG-50: Iteration Testing | Dev 3 | 3 | ✓ Done |

**Total Story Points Completed:** 11  
**Dev 1:** 3 points | **Dev 2:** 5 points | **Dev 3:** 3 points

---

**← [Day 11](./day-11.md)** | **[Day 13: Iterative Agent →](./day-13.md)**
