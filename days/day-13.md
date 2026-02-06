# Day 13: Cover Letter

> **Date:** Sprint 2, Day 13  
> **Focus:** Add cover letter generation to agent  
> **Vertical Slice:** Generate optimized cover letter from resume + JD

---

## 🎯 Today's Goal

By end of today:
- ✅ Cover letter node created (Dev 3 - LangGraph training!)
- ✅ Letter integrated into workflow (Dev 2)
- ✅ Cover letter API endpoint (Dev 1)
- ✅ User can generate cover letter with optimized resume

---

## 📋 Jira Task

### Task AG-51: Cover Letter Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-51 |
| **Title** | Cover Letter Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Create LangGraph node that generates personalized cover letter using optimized resume and job description. Dev 3 completes LangGraph training!

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/cover_letter.py`
- [ ] Uses GPT-4o-mini to generate cover letter
- [ ] Inputs: job_description, modified_resume, job_requirements
- [ ] Output: 300-400 word professional cover letter
- [ ] Addresses company and role from JD
- [ ] PR merged

---

### Task AG-52: Workflow Integration

| Field | Value |
|-------|-------|
| **Task ID** | AG-52 |
| **Title** | Workflow Integration |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Add cover letter node to workflow graph as optional final step after modification node.

**Acceptance Criteria:**
- [ ] Add cover_letter node to workflow
- [ ] Edge: modification → cover_letter (optional)
- [ ] Add `generate_cover_letter` flag to state
- [ ] Cover letter only runs if flag is True
- [ ] Updated workflow compiles successfully
- [ ] PR merged

---

### Task AG-53: Cover Letter API Endpoint

| Field | Value |
|-------|-------|
| **Task ID** | AG-53 |
| **Title** | Cover Letter API Endpoint |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | Medium |

**Description:**
Update agent API to support cover letter generation as optional parameter. Return cover letter in response.

**Acceptance Criteria:**
- [ ] Add `generate_cover_letter` param to OptimizationRequest
- [ ] Pass flag to workflow
- [ ] Include cover_letter in OptimizationResponse
- [ ] Save cover letter to database (agent_runs table)
- [ ] API docs updated
- [ ] PR merged

---

## 👥 Dev 1 + Dev 2 (PAIR): AG-15

### Step 1: Create Branch

```bash
git checkout dev-shabas && git pull origin main
git checkout -b feature/AG-15-conditional-edges
```

### Step 2: Create Decision Function

Create file: `backend/agent/nodes/decision.py`

```python
"""
Decision Node

Decides whether to iterate again or finish.
"""
from typing import Dict, Literal

from ..state import ResumeAgentState


# Target score threshold
DEFAULT_TARGET_SCORE = 70
MIN_IMPROVEMENT = 5  # Must improve by at least 5 points per iteration


def should_iterate(state: ResumeAgentState) -> Literal["iterate", "finish"]:
    """
    Decision function for conditional edge.
    
    Returns:
        "iterate" - Go back to planning for another round
        "finish" - End the workflow
    """
    current_score = state.get("ats_score_after", 0)
    iteration = state.get("iteration_count", 0)
    max_iterations = state.get("max_iterations", 3)
    target_score = DEFAULT_TARGET_SCORE
    
    # Log the decision
    decision_reason = ""
    
    # Check if we've hit max iterations
    if iteration >= max_iterations:
        decision_reason = f"Max iterations ({max_iterations}) reached"
        result = "finish"
    
    # Check if we've hit target score
    elif current_score >= target_score:
        decision_reason = f"Target score ({target_score}) achieved: {current_score}"
        result = "finish"
    
    # Check if we're making progress
    elif iteration > 1:
        history = state.get("score_history", [])
        if len(history) >= 2:
            last_improvement = history[-1] - history[-2]
            if last_improvement < MIN_IMPROVEMENT:
                decision_reason = f"Improvement too small ({last_improvement:.1f} < {MIN_IMPROVEMENT})"
                result = "finish"
            else:
                decision_reason = f"Score {current_score} < target {target_score}, iterating"
                result = "iterate"
        else:
            result = "iterate"
            decision_reason = "Continuing iterations"
    
    else:
        decision_reason = f"Score {current_score} < target {target_score}, iterating"
        result = "iterate"
    
    return result


def log_decision(state: ResumeAgentState) -> Dict:
    """
    Node to log the iteration decision.
    Called before the conditional edge routes.
    """
    current_score = state.get("ats_score_after", 0)
    iteration = state.get("iteration_count", 0)
    
    decision = should_iterate(state)
    
    log_entry = {
        "node": "iteration_decision",
        "action": decision,
        "iteration": iteration,
        "current_score": current_score,
        "target_score": DEFAULT_TARGET_SCORE
    }
    
    # Mark as final if finishing
    final_status = "completed" if decision == "finish" else "iterating"
    
    return {
        "decision_log": state.get("decision_log", []) + [log_entry],
        "final_status": final_status
    }
```

### Step 3: Update Workflow

Update file: `backend/agent/workflow.py`

```python
"""
Resume Optimization Agent Workflow

Now with iterative refinement!
"""
from langgraph.graph import StateGraph, END

from .state import ResumeAgentState
from .nodes.job_requirements import extract_job_requirements
from .nodes.resume_analysis import analyze_resume
from .nodes.scoring import score_initial, score_modified
from .nodes.planning import plan_improvements
from .nodes.modification import modify_resume
from .nodes.decision import should_iterate, log_decision


def create_agent_workflow() -> StateGraph:
    """Build the LangGraph workflow with iteration support."""
    
    workflow = StateGraph(ResumeAgentState)
    
    # Add all nodes
    workflow.add_node("extract_requirements", extract_job_requirements)
    workflow.add_node("analyze_resume", analyze_resume)
    workflow.add_node("score_initial", score_initial)
    workflow.add_node("plan_improvements", plan_improvements)
    workflow.add_node("modify_resume", modify_resume)
    workflow.add_node("score_modified", score_modified)
    workflow.add_node("log_decision", log_decision)
    
    # Initial flow (same as before)
    workflow.set_entry_point("extract_requirements")
    workflow.add_edge("extract_requirements", "analyze_resume")
    workflow.add_edge("analyze_resume", "score_initial")
    workflow.add_edge("score_initial", "plan_improvements")
    workflow.add_edge("plan_improvements", "modify_resume")
    workflow.add_edge("modify_resume", "score_modified")
    
    # After scoring, log the decision
    workflow.add_edge("score_modified", "log_decision")
    
    # Conditional edge: iterate or finish
    workflow.add_conditional_edges(
        "log_decision",
        should_iterate,
        {
            "iterate": "plan_improvements",  # Loop back
            "finish": END
        }
    )
    
    return workflow.compile()


agent_app = create_agent_workflow()


def run_optimization(
    job_description: str,
    resume: str,
    user_id: str = "anonymous",
    run_id: str = None,
    max_iterations: int = 3
) -> dict:
    """Run the full optimization with iteration support."""
    import uuid
    
    if run_id is None:
        run_id = str(uuid.uuid4())
    
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
        "max_iterations": max_iterations,
        "final_status": "pending"
    }
    
    result = agent_app.invoke(initial_state)
    result["final_status"] = "completed"
    
    return result
```

### Step 4: Test Iteration

```bash
cd backend/agent
python test_workflow.py

# Should see multiple iterations in decision log if score < 70
```

### Step 5: Commit and PR

```bash
git add backend/agent/
git commit -m "AG-15: Add conditional edges for iteration

- Create decision node to check score vs target
- Add conditional edge to loop back or finish
- Log each iteration decision
- Default max 3 iterations

Co-authored-by: Sinan <sinan@example.com>"

git push origin feature/AG-15-conditional-edges
```

---

## ✅ Day 13: Manual Verification

Run workflow and check:
1. Decision log shows iteration decisions
2. Multiple score entries in score_history
3. Stops at target score OR max iterations

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-51: Cover Letter Node | Dev 3 | 5 | ✓ Done |
| AG-52: Workflow Integration | Dev 2 | 2 | ✓ Done |
| AG-53: Cover Letter API Endpoint | Dev 1 | 3 | ✓ Done |

**Total Story Points Completed:** 10  
**Dev 1:** 3 points | **Dev 2:** 2 points | **Dev 3:** 5 points

---

**← [Day 12](./day-12.md)** | **[Day 14: Cover Letter Node →](./day-14.md)**
