# Day 14: History Backend

> **Date:** Sprint 2, Day 14  
> **Focus:** Build run history backend  
> **Vertical Slice:** Save and retrieve past optimizations

---

## 🎯 Today's Goal

By end of today:
- ✅ History database schema updated (Dev 1)
- ✅ Save runs to database (Dev 2)
- ✅ History API endpoints created (Dev 3)
- ✅ User can retrieve past optimization runs

---

## 📋 Jira Task

### Task AG-48: History Database Schema

| Field | Value |
|-------|-------|
| **Task ID** | AG-48 |
| **Title** | History Database Schema |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Update agent_runs table to fully support history feature. Add indexes for efficient querying by user and timestamps.

**Acceptance Criteria:**
- [ ] Add indexes: user_id, created_at
- [ ] Add status field: pending/running/completed/failed
- [ ] Add timestamp fields: created_at, completed_at
- [ ] Migration script created
- [ ] Database updated successfully
- [ ] PR merged

---

### Task AG-49: Save Runs to Database

| Field | Value |
|-------|-------|
| **Task ID** | AG-49 |
| **Title** | Save Runs to Database |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Update agent endpoint to save all optimization runs to database with full details (inputs, outputs, scores, timestamps).

**Acceptance Criteria:**
- [ ] Create AgentRun record on optimization start
- [ ] Save all inputs (JD, resume)
- [ ] Save all outputs (scores, modified resume, cover letter)
- [ ] Update status: pending → running → completed/failed
- [ ] Set timestamps appropriately
- [ ] PR merged

---

### Task AG-50: History API Endpoints

| Field | Value |
|-------|-------|
| **Task ID** | AG-50 |
| **Title** | History API Endpoints |
| **Type** | Task |
| **Epic** | Backend API & Database |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 4 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Create API endpoints to retrieve user's optimization history. List view + detail view for past runs.

**Acceptance Criteria:**
- [ ] GET /api/agent/history - list user's runs (paginated)
- [ ] GET /api/agent/run/:id - get full run details
- [ ] Requires authentication
- [ ] Returns runs sorted by date (newest first)
- [ ] Pagination support (limit, offset)
- [ ] PR merged

---

## 👨‍💻 Dev 3: AG-11 - Cover Letter Node

### Step 1: Create Branch

```bash
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-11-cover-letter
```

### Step 2: Create Cover Letter Node

Create file: `backend/agent/nodes/cover_letter.py`

```python
"""
Cover Letter Generation Node

Generates a personalized cover letter based on:
- Job requirements
- Resume analysis
- Modified resume content
"""
import os
from typing import Dict

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
import json

from ..state import ResumeAgentState


COVER_LETTER_PROMPT = """Write a professional cover letter for this job application.

Job Requirements:
{job_requirements}

Candidate's Background:
{resume_analysis}

Key Skills to Highlight:
{key_skills}

Instructions:
1. Open with enthusiasm for the specific role
2. Connect candidate's experience to job requirements
3. Highlight 2-3 specific achievements
4. Show knowledge of the company/role
5. End with a call to action
6. Keep it under 400 words
7. Be professional but personable

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
git commit -m "AG-11: Add cover letter generation node

- Create LLM-based cover letter generation
- Integrate into workflow after final score
- Highlight matching skills"

git push origin feature/AG-11-cover-letter
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
| AG-48: History Database Schema | Dev 1 | 3 | ✓ Done |
| AG-49: Save Runs to Database | Dev 2 | 3 | ✓ Done |
| AG-50: History API Endpoints | Dev 3 | 4 | ✓ Done |

**Total Story Points Completed:** 10  
**Dev 1:** 3 points | **Dev 2:** 3 points | **Dev 3:** 4 points

---

**← [Day 13](./day-13.md)** | **[Day 15: History UI →](./day-15.md)**
