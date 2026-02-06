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

### Task AG-22: User Database Model

| Field | Value |
|-------|-------|
| **Task ID** | AG-22 |
| **Title** | User Database Model |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create Supabase database schema for users table with email, hashed password, and timestamps. This is the foundation for auth.

**Acceptance Criteria:**
- [ ] Supabase migration created for users table
- [ ] Fields: id, email, hashed_password, created_at
- [ ] Email is unique and indexed
- [ ] Migration runs successfully
- [ ] PR merged

---

### Task AG-23: Registration Endpoint

| Field | Value |
|-------|-------|
| **Task ID** | AG-23 |
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

### Task AG-24: Login & JWT Endpoint

| Field | Value |
|-------|-------|
| **Task ID** | AG-24 |
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

### Task AG-25: Auth Middleware

| Field | Value |
|-------|-------|
| **Task ID** | AG-25 |
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

### Step 1: Setup (Dev 1 creates branch)

```bash
cd resume-agent
git checkout dev-shabas && git pull origin main
git checkout -b feature/AG-10-workflow-assembly
```

**Jira:** Move AG-10 to "In Progress"

### Step 2: Create Workflow File

Create file: `backend/agent/workflow.py`

```python
"""
Resume Optimization Agent Workflow

This is the main LangGraph workflow that orchestrates all nodes.

Flow:
1. extract_requirements - Parse job description
2. analyze_resume - Analyze current resume
3. score_initial - Get baseline ATS score
4. plan_improvements - Create modification plan
5. modify_resume - Apply improvements
6. score_modified - Measure improvement
7. END

Sprint 1: Linear flow (no decision gates)
Sprint 2: Add conditional edges for iteration
"""
from langgraph.graph import StateGraph, END

from .state import ResumeAgentState
from .nodes.job_requirements import extract_job_requirements
from .nodes.resume_analysis import analyze_resume
from .nodes.scoring import score_initial, score_modified
from .nodes.planning import plan_improvements
from .nodes.modification import modify_resume


def create_agent_workflow() -> StateGraph:
    """
    Build and return the compiled LangGraph workflow.
    
    Returns:
        Compiled workflow that can process resume optimization requests
    """
    # Initialize workflow with our state type
    workflow = StateGraph(ResumeAgentState)
    
    # === Add all nodes ===
    workflow.add_node("extract_requirements", extract_job_requirements)
    workflow.add_node("analyze_resume", analyze_resume)
    workflow.add_node("score_initial", score_initial)
    workflow.add_node("plan_improvements", plan_improvements)
    workflow.add_node("modify_resume", modify_resume)
    workflow.add_node("score_modified", score_modified)
    
    # === Define the flow (linear for Sprint 1) ===
    
    # Start: Extract requirements from job description
    workflow.set_entry_point("extract_requirements")
    
    # Then: Analyze the resume
    workflow.add_edge("extract_requirements", "analyze_resume")
    
    # Then: Score the original resume
    workflow.add_edge("analyze_resume", "score_initial")
    
    # Then: Create improvement plan
    workflow.add_edge("score_initial", "plan_improvements")
    
    # Then: Apply modifications
    workflow.add_edge("plan_improvements", "modify_resume")
    
    # Then: Score the modified version
    workflow.add_edge("modify_resume", "score_modified")
    
    # Finally: End
    workflow.add_edge("score_modified", END)
    
    # Compile and return
    return workflow.compile()


# Create the compiled app for import
agent_app = create_agent_workflow()


def run_optimization(
    job_description: str,
    resume: str,
    user_id: str = "anonymous",
    run_id: str = None
) -> dict:
    """
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

git add backend/agent/

git commit -m "AG-10: Assemble complete LangGraph workflow

- Connect all nodes: extract → analyze → score → plan → modify → score
- Create workflow.py with StateGraph configuration
- Add run_optimization convenience function
- Create comprehensive end-to-end tests
- Verify score improvement works!

Team effort: Shabas, Sinan, Marva"

git push origin feature/AG-10-workflow-assembly
```

**Jira:** Move AG-10 to "In Review"

### Step 7: Open PR and Merge

PR Title: `AG-10: Complete LangGraph Workflow Assembly`

After review and merge:
```bash
git checkout dev-shabas
git pull origin main
git branch -d feature/AG-10-workflow-assembly
```

**Jira:** Move AG-10 to "Done"

---

## ✅ Day 6: Manual Verification (CRITICAL!)

### Terminal Tests

```bash
cd backend/agent

# Run full workflow test
python test_workflow.py

# Verify all 6 nodes executed
# Verify score improved
# Verify decision log has entries
```

### What Must Work

| Check | Expected |
|-------|----------|
| Workflow runs without errors | ✓ |
| Job requirements extracted | ✓ |
| Resume analyzed | ✓ |
| Initial score calculated | ✓ |
| Improvement plan created | ✓ |
| Resume modified | ✓ |
| Final score calculated | ✓ |
| Score improved (delta > 0) | ✓ |

### If Something Fails

Common issues:
1. **API Key not set** → Check `.env` file
2. **Import errors** → Check all `__init__.py` files
3. **JSON parse errors** → Check LLM prompts return valid JSON
4. **Score not improving** → Review scoring function weights

---

## 📁 Folder Structure After Day 6

```
backend/agent/
├── __init__.py
├── state.py              ✓
├── scoring.py            ✓
├── workflow.py           ✓ NEW - Main workflow!
├── test_workflow.py      ✓ NEW - End-to-end tests
└── nodes/
    ├── __init__.py       ✓ Updated
    ├── job_requirements.py
    ├── resume_analysis.py
    ├── scoring.py
    ├── planning.py
    └── modification.py
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-22: User Database Model | Dev 2 | 3 | ✓ Done |
| AG-23: Registration Endpoint | Dev 1 | 3 | ✓ Done |
| AG-24: Login & JWT Endpoint | Dev 2 | 3 | ✓ Done |
| AG-25: Auth Middleware | Dev 3 | 3 | ✓ Done |

**Total Story Points Completed:** 12  
**Dev 1:** 3 points | **Dev 2:** 6 points | **Dev 3:** 3 points

---

## 🎉 MILESTONE: Core Agent Complete!

The core agent workflow is now functional. You can:
- Input a job description + resume
- Get an optimized resume back
- See the score improvement

**Next steps:**
- Day 7-8: Connect to API
- Day 9: Show in UI
- Day 10: Sprint 1 Demo!

---

**← [Day 5](./day-05.md)** | **[Day 7: Auth API →](./day-07.md)**
