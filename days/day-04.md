# Day 4: Planning & Modification Nodes

> **Date:** Sprint 1, Day 4  
> **Focus:** Build planning & modification nodes + re-scoring  
> **Vertical Slice:** Plan generation and resume modification working

---

## 🎯 Today's Goal

By end of today:
- ✅ Improvement Planning node working (Dev 1)
- ✅ Resume Modification node working (Dev 3 - 2nd LangGraph node!)
- ✅ Modified Resume Scoring node working (Dev 2)
- ✅ Can see resume improvement in action

---

## 📋 Morning: Add These Tasks to Jira

---

### Task AG-21: Improvement Planning Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-21 |
| **Title** | Improvement Planning Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Build a LangGraph node that compares job requirements with resume analysis to create an actionable improvement plan. This node decides WHAT changes to make to optimize the resume.

**What You'll Learn:**
- Strategic planning with LLMs
- Comparing two data structures
- Generating actionable recommendations
- Decision-making in agent workflows

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/planning.py`
- [ ] Takes job_requirements and resume_analysis from state
- [ ] Creates prioritized improvement plan
- [ ] Returns specific, actionable changes
- [ ] Test file validates planning logic
- [ ] PR merged to main

**Dependencies:** AG-15 (job requirements), AG-18 (resume analysis), AG-19 (scoring)

---

### Task AG-22: Resume Modification Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-22 |
| **Title** | Resume Modification Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Build a LangGraph node that takes the improvement plan and actually modifies the resume text. This is Dev 3's SECOND LangGraph node - getting more experience! Uses GPT-4o to intelligently rewrite resume sections.

**What You'll Learn:**
- Text modification with LLMs
- Implementing planned changes 
- Preserving resume structure while improving content
- More LangGraph node patterns

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/modification.py`
- [ ] Takes improvement_plan and original_resume from state
- [ ] Returns modified resume text
- [ ] Preserves resume format
- [ ] Updates state's `modified_resume` field
- [ ] Test file shows clear improvement
- [ ] PR merged to main

**Dependencies:** AG-21 (improvement plan must exist)

**Related Tasks:** Dev 3's 2nd LangGraph node!

---

### Task AG-23: Modified Resume Scoring Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-23 |
| **Title** | Modified Resume Scoring Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Build a LangGraph node that scores the MODIFIED resume against job requirements to see if the changes improved the score. This calculates the "after" score and improvement delta.

**What You'll Learn:**
- Reusing scoring logic in new contexts
- Calculating deltas and improvements
- Updating multiple state fields in one node
- Testing score improvements

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/rescore.py`
- [ ] Scores modified resume (reuses AG-25 scoring logic)
- [ ] Updates state's `ats_score_after` field
- [ ] Calculates `improvement_delta` (after - before)
- [ ] Adds to `score_history` list
- [ ] Test shows score improvement
- [ ] PR merged to main

**Dependencies:** AG-22 (modified resume must exist), AG-19 (scoring logic)

**Related Tasks:** Shows the agent actually improved the resume!

---

## 👨‍💻 Dev 2 (Sinan): AG-18 - Resume Analysis Node

### Step 1: Create Branch

```bash
cd resume-agent
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-18-resume-analysis
```

**Jira:** Move AG-18 to "In Progress"

### Step 2: Create Resume Analysis Node

Create file: `backend/agent/nodes/resume_analysis.py`

```python
"""
Resume Analysis Node

Analyzes the user's resume to extract:
- Current skills and technologies
- Experience level and years
- Strengths and achievements
- Gaps compared to typical job requirements
"""
import json
import os
from typing import Dict

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

from ..state import ResumeAgentState


ANALYSIS_PROMPT = """Analyze this resume and extract structured information.

Resume:
---
{resume}
---

Extract and return as JSON:
1. **current_skills**: List of technical and soft skills found
2. **experience_years**: Total years of experience (number or estimate)
3. **experience_level**: "entry" | "mid" | "senior" | "lead"
4. **education**: List of degrees/certifications
5. **achievements**: Notable accomplishments with numbers/metrics
6. **strengths**: What the candidate does well
7. **gaps**: Common skills that seem missing for their level
8. **summary**: One sentence summary of the candidate

Return ONLY valid JSON, no explanation."""


def analyze_resume(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Analyze resume content
    
    Args:
        state: Current agent state with original_resume
        
    Returns:
        Dict with resume_analysis to merge into state
    """
    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0,
        api_key=os.getenv("OPENAI_API_KEY")
    )
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You analyze resumes and return structured JSON data."),
        ("user", ANALYSIS_PROMPT)
    ])
    
    chain = prompt | llm
    
    result = chain.invoke({
        "resume": state["original_resume"]
    })
    
    try:
        analysis = json.loads(result.content)
    except json.JSONDecodeError:
        analysis = {
            "current_skills": [],
            "experience_years": "unknown",
            "experience_level": "unknown",
            "strengths": [],
            "gaps": [],
            "parse_error": True
        }
    
    decision = {
        "node": "analyze_resume",
        "action": "analyzed_resume_content",
        "skills_found": len(analysis.get("current_skills", [])),
        "experience_level": analysis.get("experience_level", "unknown")
    }
    
    return {
        "resume_analysis": analysis,
        "decision_log": state.get("decision_log", []) + [decision]
    }
```

### Step 3: Create Test

Create file: `backend/agent/nodes/test_resume_analysis.py`

```python
"""Tests for resume analysis node."""
import os
from dotenv import load_dotenv
load_dotenv()

if not os.getenv("OPENAI_API_KEY"):
    print("❌ OPENAI_API_KEY not set")
    exit(1)

from resume_analysis import analyze_resume


def test_analyze_resume():
    """Test resume analysis."""
    state = {
        "run_id": "test",
        "user_id": "test",
        "job_description": "",
        "original_resume": """
        John Doe
        Senior Software Engineer | john@email.com
        
        Summary:
        7 years of experience in backend development.
        
        Skills:
        Python, Django, FastAPI, PostgreSQL, Docker, AWS
        
        Experience:
        Senior Engineer at TechCorp (2020-Present)
        - Led team of 5 developers
        - Built APIs handling 1M requests/day
        - Reduced deployment time by 60%
        
        Software Engineer at StartupXYZ (2017-2020)
        - Developed Python microservices
        - Implemented CI/CD pipelines
        
        Education:
        BS Computer Science, 2017
        """,
        "decision_log": []
    }
    
    result = analyze_resume(state)
    
    assert "resume_analysis" in result
    analysis = result["resume_analysis"]
    
    print(f"Skills found: {analysis.get('current_skills', [])}")
    print(f"Experience: {analysis.get('experience_years')} years")
    print(f"Level: {analysis.get('experience_level')}")
    
    assert len(analysis.get("current_skills", [])) > 0
    print("✓ Resume analysis test passed!")


if __name__ == "__main__":
    test_analyze_resume()
    print("\n🎉 All tests passed!")
```

### Step 4: Test, Commit, PR

```bash
cd backend/agent/nodes
python test_resume_analysis.py

cd ../../..
git add backend/agent/nodes/
git commit -m "AG-16: Create resume analysis node

- Extract skills, experience, achievements from resume
- Return structured JSON analysis
- Add decision logging"

git push origin feature/AG-16-resume-analysis
```

Open PR → Merge

**Jira:** Move AG-16 to "Done"

---

## 👥 Dev 1 + Dev 3 (PAIR): AG-19 - Planning Node

### Step 1: Setup (Dev 1 drives first)

```bash
cd resume-agent
git checkout dev-shabas && git pull origin main
git checkout -b feature/AG-19-planning-node
```

**Jira:** Move AG-19 to "In Progress"

### Step 2: Create Planning Node

Create file: `backend/agent/nodes/planning.py`

```python
"""
Improvement Planning Node

Compares job requirements with resume analysis and creates
an actionable plan for improving the resume.
"""
import json
import os
from typing import Dict

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

from ..state import ResumeAgentState


PLANNING_PROMPT = """Create a resume improvement plan.

Job Requirements:
{job_requirements}

Resume Analysis:
{resume_analysis}

Current ATS Score: {current_score}

Create a plan with:
1. **priority_changes**: List of highest-impact changes (max 5)
2. **skill_additions**: Skills to add or emphasize
3. **keyword_insertions**: Specific keywords to incorporate
4. **section_improvements**: Which sections need work
5. **expected_score_gain**: Estimated score improvement (number)
6. **reasoning**: Why these changes will help

Each change should be specific and actionable.

Return ONLY valid JSON."""


def plan_improvements(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Create improvement plan
    
    Args:
        state: Current state with analysis and requirements
        
    Returns:
        Dict with improvement_plan to merge into state
    """
    llm = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0.3,  # Slight creativity for planning
        api_key=os.getenv("OPENAI_API_KEY")
    )
    
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You create actionable resume improvement plans. Return JSON only."),
        ("user", PLANNING_PROMPT)
    ])
    
    chain = prompt | llm
    
    result = chain.invoke({
        "job_requirements": json.dumps(state.get("job_requirements", {})),
        "resume_analysis": json.dumps(state.get("resume_analysis", {})),
        "current_score": state.get("ats_score_before", "Not yet scored")
    })
    
    try:
        plan = json.loads(result.content)
    except json.JSONDecodeError:
        plan = {
            "priority_changes": [],
            "skill_additions": [],
            "expected_score_gain": 0,
            "parse_error": True
        }
    
    decision = {
        "node": "plan_improvements",
        "action": "created_improvement_plan",
        "changes_planned": len(plan.get("priority_changes", [])),
        "expected_gain": plan.get("expected_score_gain", 0)
    }
    
    return {
        "improvement_plan": plan,
        "decision_log": state.get("decision_log", []) + [decision]
    }
```

**⏰ SWITCH DRIVERS at 45 min mark**

### Step 3: Create Test

Create file: `backend/agent/nodes/test_planning.py`

```python
"""Tests for planning node."""
import os
from dotenv import load_dotenv
load_dotenv()

from planning import plan_improvements


def test_create_plan():
    """Test improvement planning."""
    state = {
        "run_id": "test",
        "user_id": "test",
        "job_description": "Python developer with Django",
        "original_resume": "John Doe, Python developer",
        "job_requirements": {
            "must_have_skills": ["Python", "Django", "PostgreSQL"],
            "keywords": ["backend", "API", "scalable"]
        },
        "resume_analysis": {
            "current_skills": ["Python", "Flask"],
            "gaps": ["Django", "PostgreSQL"]
        },
        "ats_score_before": 45,
        "decision_log": []
    }
    
    result = plan_improvements(state)
    
    assert "improvement_plan" in result
    plan = result["improvement_plan"]
    
    print(f"Priority changes: {plan.get('priority_changes', [])}")
    print(f"Expected gain: {plan.get('expected_score_gain')}")
    
    print("✓ Planning test passed!")


if __name__ == "__main__":
    test_create_plan()
    print("\n🎉 All tests passed!")
```

### Step 4: Test, Commit, PR

```bash
cd backend/agent/nodes
python test_planning.py

cd ../../..
git add backend/agent/nodes/
git commit -m "AG-19: Create improvement planning node

- Compare requirements vs resume analysis
- Generate prioritized improvement plan
- Calculate expected score gain

Co-authored-by: Marva <marva@example.com>"

git push origin feature/AG-19-planning-node
```

Open PR → Merge

**Jira:** Move AG-19 to "Done"

---

## ✅ Day 4: Manual Verification

### Terminal Tests

```bash
# Test resume analysis
cd backend/agent/nodes
python test_resume_analysis.py
# Expected: Skills extracted, experience level identified

# Test planning
python test_planning.py
# Expected: Improvement plan with priority changes
```

### What to Verify
1. Resume analysis extracts skills correctly
2. Planning node creates actionable changes
3. Both return valid JSON
4. Decision logs are updated

---

## 📁 Folder Structure After Day 4

```
backend/agent/nodes/
├── job_requirements.py      ✓ Day 3
├── resume_analysis.py       ✓ NEW
├── planning.py              ✓ NEW
├── test_*.py files          ✓ Tests
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-21: Improvement Planning Node | Dev 1 | 5 | ✓ Done |
| AG-22: Resume Modification Node | Dev 3 | 5 | ✓ Done |
| AG-23: Modified Resume Scoring Node | Dev 2 | 3 | ✓ Done |

**Total Story Points Completed:** 13  
**Dev 1:** 5 points | **Dev 2:** 3 points | **Dev 3:** 5 points

---

**← [Day 3](./day-03.md)** | **[Day 5: Modification Node →](./day-05.md)**
