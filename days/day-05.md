# Day 5: Workflow Assembly

> **Date:** Sprint 1, Day 5  
> **Focus:** Connect ALL nodes into complete workflow + testing  
> **Vertical Slice:** COMPLETE working agent end-to-end!

---

## 🎯 Today's Goal

By end of today:
- ✅ All 6 LangGraph nodes connected in workflow (Dev 1)
- ✅ State flows correctly through all nodes (Dev 2)
- ✅ End-to-end agent test works (Dev 3)
- ✅ Score history tracking implemented (Dev 2)
- ✅ FIRST COMPLETE AGENT WORKFLOW RUNNING!

---

## 📋 Morning: Add These Tasks to Jira

---

### Task AG-18: Workflow Graph Construction

| Field | Value |
|-------|-------|
| **Task ID** | AG-18 |
| **Title** | Workflow Graph Construction |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Connect ALL 6 LangGraph nodes into the complete agent workflow graph. This creates the first end-to-end working agent! Nodes: Extract Requirements → Analyze Resume → Score Initial → Plan Improvements → Modify Resume → Score Modified.

**What You'll Learn:**
- Building complex multi-node workflows
- Designing optimal node execution order
- Complete StateGraph assembly
- Testing full agent workflows

**Acceptance Criteria:**
- [ ] File created: `backend/agent/workflow.py`
- [ ] All 6 nodes connected with edges
- [ ] Entry point set to extract_requirements
- [ ] Workflow compiles without errors
- [ ] Can invoke with job description + resume
- [ ] Test file shows all nodes executing
- [ ] PR merged to main

**Dependencies:** AG-9, AG-12, AG-13, AG-15, AG-16, AG-17 (all nodes exist)

---

### Task AG-19: State Flow Testing

| Field | Value |
|-------|-------|
| **Task ID** | AG-19 |
| **Title** | State Flow Testing |
| **Type** | Task |
| **Epic** | Testing |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create comprehensive tests that validate state flows correctly through all 6 nodes. Verify each node updates the correct state fields and decision log is populated.

**What You'll Learn:**
- Testing stateful workflows
- Validating state transformations
- Debugging multi-node flows
- Agent testing patterns

**Acceptance Criteria:**
- [ ] File created: `backend/agent/test_state_flow.py`
- [ ] Tests verify all state fields are populated
- [ ] Tests check decision_log has 6+ entries
- [ ] Tests validate score improvement
- [ ] All tests pass
- [ ] PR merged to main

**Dependencies:** AG-18 (workflow must be assembled)

---

### Task AG-20: End-to-End Agent Test

| Field | Value |
|-------|-------|
| **Task ID** | AG-20 |
| **Title** | End-to-End Agent Test |
| **Type** | Task |
| **Epic** | Testing |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a real-world end-to-end test with actual job description and resume. Run the complete workflow and verify it produces improved results.

**What You'll Learn:**
- End-to-end testing strategies
- Validating real agent outputs
- Quality assurance for AI systems

**Acceptance Criteria:**
- [ ] File created: `backend/agent/test_e2e.py`
- [ ] Uses realistic sample job description and resume
- [ ] Runs complete workflow from start to finish
- [ ] Validates final score > initial score
- [ ] Prints before/after comparison
- [ ] Test passes successfully
- [ ] PR merged to main

**Dependencies:** AG-18 (workflow must exist)

---

### Task AG-21: Score History Tracking

| Field | Value |
|-------|-------|
| **Task ID** | AG-21 |
| **Title** | Score History Tracking |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | Medium |

**Description:**
Add functionality to track score changes across iterations. This will be useful later for iterative improvements and showing progress to users.

**What You'll Learn:**
- State field management
- Historical data tracking
- Score progression visualization prep

**Acceptance Criteria:**
- [ ] score_history field properly maintained
- [ ] Each scoring node appends to history
- [ ] Test validates history list grows
- [ ] PR merged to main

**Dependencies:** AG-18, AG-19

---

## 👨‍💻 Dev 3 (Marva): AG-8 - Modification Node

### Step 1: Create Branch

```bash
cd resume-agent
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-8-modification-node
```

**Jira:** Move AG-8 to "In Progress"

### Step 2: Create Modification Node

Create file: `backend/agent/nodes/modification.py`

```python
"""
Resume Modification Node

Takes the improvement plan and applies changes to the resume.
Uses LLM to rewrite while maintaining the candidate's authentic voice.
"""
import os
from typing import Dict

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

from ..state import ResumeAgentState


MODIFICATION_PROMPT = """Improve this resume based on the plan provided.

Original Resume:
---
{original_resume}
---

Improvement Plan:
{improvement_plan}

Job Requirements (for context):
{job_requirements}

Instructions:
1. Apply the priority changes from the plan
2. Add the recommended skills naturally
3. Incorporate keywords without keyword stuffing
4. Keep the candidate's authentic voice
5. Maintain truthfulness - don't add fake experience
6. Use strong action verbs
7. Quantify achievements where possible

Return the COMPLETE modified resume text. Make it natural and professional."""


def modify_resume(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Modify resume according to plan
    
    Args:
        state: Current state with resume and improvement plan
        
    Returns:
        Dict with modified_resume to merge into state
    """
    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0.4,  # Balance between creativity and consistency
        api_key=os.getenv("OPENAI_API_KEY")
    )
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", """You are an expert resume writer. You improve resumes while 
        keeping them authentic and truthful. Never add fake experience or skills 
        the candidate doesn't have - only enhance presentation of existing content."""),
        ("user", MODIFICATION_PROMPT)
    ])
    
    chain = prompt | llm
    
    import json
    result = chain.invoke({
        "original_resume": state["original_resume"],
        "improvement_plan": json.dumps(state.get("improvement_plan", {})),
        "job_requirements": json.dumps(state.get("job_requirements", {}))
    })
    
    modified_resume = result.content
    
    # Track what was done
    plan = state.get("improvement_plan", {})
    changes_applied = plan.get("priority_changes", [])
    
    decision = {
        "node": "modify_resume",
        "action": "applied_improvements",
        "changes_count": len(changes_applied),
        "original_length": len(state["original_resume"]),
        "modified_length": len(modified_resume)
    }
    
    return {
        "modified_resume": modified_resume,
        "applied_changes": changes_applied,
        "iteration_count": state.get("iteration_count", 0) + 1,
        "decision_log": state.get("decision_log", []) + [decision]
    }
```

### Step 3: Create Test

Create file: `backend/agent/nodes/test_modification.py`

```python
"""Tests for modification node."""
import os
from dotenv import load_dotenv
load_dotenv()

from modification import modify_resume


def test_modify_resume():
    """Test resume modification."""
    state = {
        "run_id": "test",
        "user_id": "test",
        "job_description": "Python backend developer",
        "original_resume": """
        John Doe
        Software Developer
        
        Experience:
        Developer at Company (2020-Present)
        - Built web applications
        - Worked with databases
        
        Skills:
        Python, JavaScript
        """,
        "job_requirements": {
            "must_have_skills": ["Python", "Django", "PostgreSQL"],
            "keywords": ["backend", "API", "scalable"]
        },
        "resume_analysis": {
            "current_skills": ["Python", "JavaScript"],
            "gaps": ["Django", "PostgreSQL"]
        },
        "improvement_plan": {
            "priority_changes": [
                "Add more detail to work experience",
                "Include backend/API keywords",
                "Quantify achievements"
            ],
            "keyword_insertions": ["backend", "API", "scalable"],
            "expected_score_gain": 15
        },
        "iteration_count": 0,
        "decision_log": []
    }
    
    result = modify_resume(state)
    
    assert "modified_resume" in result
    modified = result["modified_resume"]
    
    print("=== ORIGINAL ===")
    print(state["original_resume"][:200])
    print("\n=== MODIFIED ===")
    print(modified[:400])
    
    # Check modifications were made
    assert len(modified) > len(state["original_resume"]) * 0.8  # Not dramatically shorter
    assert result["iteration_count"] == 1
    
    print("\n✓ Modification test passed!")


if __name__ == "__main__":
    test_modify_resume()
    print("\n🎉 All tests passed!")
```

### Step 4: Test, Commit, PR

```bash
cd backend/agent/nodes
python test_modification.py

cd ../../..
git add backend/agent/nodes/
git commit -m "AG-8: Create resume modification node

- Apply improvement plan to resume
- Maintain authentic voice
- Track iteration count and changes applied"

git push origin feature/AG-8-modification-node
```

**Jira:** Move AG-8 to "Done"

---

## 👨‍💻 Dev 2 (Sinan): AG-6 - Scoring Nodes

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-6-scoring-nodes
```

**Jira:** Move AG-6 to "In Progress"

### Step 2: Create Scoring Nodes

Create file: `backend/agent/nodes/scoring.py`

```python
"""
Scoring Nodes

LangGraph nodes that wrap the ATS scoring function.
- score_initial: Score the original resume
- score_modified: Score the modified resume
"""
from typing import Dict

from ..scoring import calculate_ats_score
from ..state import ResumeAgentState


def score_initial(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Score the original resume
    
    Called early in workflow to establish baseline.
    """
    requirements = state.get("job_requirements", {})
    
    result = calculate_ats_score(
        state["original_resume"],
        requirements
    )
    
    score = result["score"]
    
    decision = {
        "node": "score_initial",
        "action": "calculated_initial_score",
        "score": score,
        "breakdown": result["breakdown"]
    }
    
    return {
        "ats_score_before": score,
        "score_history": [score],
        "decision_log": state.get("decision_log", []) + [decision]
    }


def score_modified(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Score the modified resume
    
    Called after modification to measure improvement.
    """
    requirements = state.get("job_requirements", {})
    
    # Score the modified version
    result = calculate_ats_score(
        state.get("modified_resume", state["original_resume"]),
        requirements
    )
    
    score = result["score"]
    before_score = state.get("ats_score_before", 0)
    improvement = score - before_score
    
    # Update score history
    history = state.get("score_history", []).copy()
    history.append(score)
    
    decision = {
        "node": "score_modified",
        "action": "calculated_modified_score",
        "score": score,
        "improvement": improvement,
        "iteration": state.get("iteration_count", 1)
    }
    
    return {
        "ats_score_after": score,
        "improvement_delta": improvement,
        "score_history": history,
        "decision_log": state.get("decision_log", []) + [decision]
    }
```

### Step 3: Create Test

Create file: `backend/agent/nodes/test_scoring_nodes.py`

```python
"""Tests for scoring nodes."""
from scoring import score_initial, score_modified


def test_score_initial():
    """Test initial scoring."""
    state = {
        "run_id": "test",
        "user_id": "test",
        "job_description": "",
        "original_resume": """
        Python developer with Django and PostgreSQL experience.
        Built scalable backend APIs.
        Experience: 5 years
        Skills: Python, Django, PostgreSQL, Docker
        Education: BS Computer Science
        """,
        "job_requirements": {
            "must_have_skills": ["Python", "Django", "PostgreSQL"],
            "keywords": ["backend", "API", "scalable"]
        },
        "decision_log": []
    }
    
    result = score_initial(state)
    
    assert "ats_score_before" in result
    assert "score_history" in result
    
    print(f"Initial score: {result['ats_score_before']}")
    assert result["ats_score_before"] > 50  # Should match well
    
    print("✓ Score initial test passed!")


def test_score_modified():
    """Test modified scoring."""
    state = {
        "run_id": "test",
        "user_id": "test",
        "job_description": "",
        "original_resume": "Basic resume",
        "modified_resume": """
        Python developer with Django and PostgreSQL experience.
        Built scalable backend REST APIs handling millions of requests.
        Experience: 5 years in software development
        Skills: Python, Django, FastAPI, PostgreSQL, Docker, Kubernetes
        Education: BS Computer Science
        Summary: Experienced backend engineer
        """,
        "job_requirements": {
            "must_have_skills": ["Python", "Django", "PostgreSQL"],
            "keywords": ["backend", "API", "scalable"]
        },
        "ats_score_before": 40,
        "score_history": [40],
        "iteration_count": 1,
        "decision_log": []
    }
    
    result = score_modified(state)
    
    assert "ats_score_after" in result
    assert "improvement_delta" in result
    
    print(f"Modified score: {result['ats_score_after']}")
    print(f"Improvement: {result['improvement_delta']}")
    
    assert result["improvement_delta"] > 0  # Should improve
    
    print("✓ Score modified test passed!")


if __name__ == "__main__":
    test_score_initial()
    print()
    test_score_modified()
    print("\n🎉 All tests passed!")
```

### Step 4: Commit and PR

```bash
cd backend/agent/nodes
python test_scoring_nodes.py

cd ../../..
git add backend/agent/nodes/
git commit -m "AG-6: Create scoring nodes for workflow

- score_initial for baseline score
- score_modified for post-change score
- Track improvement delta and history"

git push origin feature/AG-6-scoring-nodes
```

**Jira:** Move AG-6 to "Done"

---

## ✅ Day 5: Manual Verification

### Terminal Tests

```bash
cd backend/agent/nodes

# Test modification
python test_modification.py
# Expected: Resume modified with improvements

# Test scoring nodes
python test_scoring_nodes.py
# Expected: Initial and modified scores calculated
```

### Integration Check

```bash
# All nodes should now exist:
ls -la backend/agent/nodes/*.py
# Should show: job_requirements.py, resume_analysis.py, planning.py, modification.py, scoring.py
```

---

## 📁 Folder Structure After Day 5

```
backend/agent/nodes/
├── __init__.py
├── job_requirements.py   ✓ Day 3
├── resume_analysis.py    ✓ Day 4
├── planning.py           ✓ Day 4
├── modification.py       ✓ NEW
├── scoring.py            ✓ NEW
└── test_*.py             ✓ Tests
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-18: Workflow Graph Construction | Dev 1 | 5 | ✓ Done |
| AG-19: State Flow Testing | Dev 2 | 3 | ✓ Done |
| AG-20: End-to-End Agent Test | Dev 3 | 3 | ✓ Done |
| AG-21: Score History Tracking | Dev 2 | 2 | ✓ Done |

**Total Story Points Completed:** 13  
**Dev 1:** 5 points | **Dev 2:** 5 points | **Dev 3:** 3 points

**🎉 MILESTONE: First complete agent workflow working end-to-end!**

---

**← [Day 4](./day-04.md)** | **[Day 6: Workflow Assembly →](./day-06.md)**
