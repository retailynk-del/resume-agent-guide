# Day 13: Cover Letter Generation

> **Date:** Sprint 2, Day 13  
> **Focus:** Add cover letter generation to agent  
> **Vertical Slice:** Generate optimized cover letter from resume + JD

---

## 🎯 Today's Goal

By end of today:
- ✅ Cover letter node created (Dev 3 - completes LangGraph training!)
- ✅ Letter integrated into workflow (Dev 2)
- ✅ Cover letter API endpoint (Dev 1)
- ✅ User can generate cover letter with optimized resume

---

## 📋 Jira Tasks

### Task AG-51: Cover Letter Generation Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-51 |
| **Title** | Cover Letter Generation Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |
| **Priority** | High |
| **Dependencies** | AG-50 (Workflow complete) |

**Description:**
Create LangGraph node that generates personalized cover letter using optimized resume and job description. This completes Dev 3's LangGraph node training - now all 3 devs have created nodes!

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/generate_cover_letter.py`
- [ ] Uses GPT-4o-mini to generate cover letter
- [ ] Inputs: job_description, modified_resume, job_requirements
- [ ] Output: 300-400 word professional cover letter
- [ ] Addresses company and role from JD
- [ ] Follows standard cover letter structure
- [ ] Test coverage > 80%
- [ ] PR approved and merged

---

### Task AG-52: Cover Letter Workflow Integration

| Field | Value |
|-------|-------|
| **Task ID** | AG-52 |
| **Title** | Cover Letter Workflow Integration |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | High |
| **Dependencies** | AG-51 |

**Description:**
Add cover letter node to workflow graph as optional final step after score_modified node. User can request cover letter generation via flag.

**Acceptance Criteria:**
- [ ] Add generate_cover_letter node to workflow
- [ ] Add `generate_cover_letter` boolean flag to state
- [ ] Conditional edge: if flag True → generate_cover_letter, else → END
- [ ] Cover letter only generated when requested
- [ ] Updated workflow compiles successfully
- [ ] Integration test passes
- [ ] PR approved and merged

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
| **Dependencies** | AG-52 |

**Description:**
Update agent API to support cover letter generation as optional parameter. Store cover letter in database and return in response.

**Acceptance Criteria:**
- [ ] Add `generate_cover_letter` optional param to API
- [ ] Pass flag to workflow
- [ ] Add `cover_letter` column to agent_runs table
- [ ] Save cover letter to database
- [ ] Include cover_letter in response
- [ ] API docs updated
- [ ] Tests pass
- [ ] PR approved and merged

---

## 🕘 9:00 AM - Daily Standup

**Shabas:** "I'll update the API to accept the cover letter flag today."

**Sinan:** "I'll integrate Marva's cover letter node into the workflow with conditional routing."

**Marva:** "I'm excited to build the cover letter node! It's my third LangGraph node - completing my training!"

---

## 👩‍💻 Dev 3 (Marva): AG-51 - Cover Letter Generation Node

### Step 1: Create Branch

```bash
git checkout marva-Dev && git pull origin main
git checkout -b feature/AG-51-cover-letter-node
```

**Jira:** Move AG-51 to "In Progress"

### Step 2: Create Cover Letter Node

Create `backend/agent/nodes/generate_cover_letter.py`:

```python
"""
Cover Letter Generation Node

Generates personalized cover letter from optimized resume and job description.
"""
from typing import Dict
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate


# Initialize LLM
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7  # Slightly creative for personalization
)


COVER_LETTER_PROMPT = """You are an expert career advisor and cover letter writer.

Generate a professional, personalized cover letter based on:

**Job Description:**
{job_description}

**Candidate's Optimized Resume:**
{modified_resume}

**Job Requirements:**
Must-have skills: {must_have_skills}
Nice-to-have skills: {nice_to_have_skills}
Key responsibilities: {key_phrases}

**Instructions:**
1. Extract company name and position title from job description
2. Create opening paragraph showing enthusiasm and fit
3. Highlight 2-3 most relevant experiences from resume
4. Connect candidate's skills directly to job requirements
5. Show understanding of company's needs
6. Close with call to action
7. Keep to 300-400 words
8. Use professional but warm tone
9. Avoid generic phrases like "I am writing to apply..."
10. Make it feel personalized, not templated

**Format:**
Dear Hiring Manager,

[Opening paragraph - Hook + position interest]

[Body paragraph 1 - Relevant experience #1]

[Body paragraph 2 - Relevant experience #2 or skills alignment]

[Closing paragraph - Enthusiasm + call to action]

Sincerely,
[Candidate Name from Resume]

Generate the cover letter:"""


def generate_cover_letter(state: Dict) -> Dict:
    """
    Generate personalized cover letter.
    
    Args:
        state: Must contain:
            - job_description: str
            - modified_resume: str
            - job_requirements: dict with must_have_skills, nice_to_have_skills, keywords
    
    Returns:
        dict with:
            - cover_letter: str (generated letter)
    """
    print("\n🎯 Generating cover letter...")
    
    job_description = state.get("job_description", "")
    modified_resume = state.get("modified_resume", "")
    job_requirements = state.get("job_requirements", {})
    
    # Extract requirements
    must_have = job_requirements.get("must_have_skills", [])
    nice_to_have = job_requirements.get("nice_to_have_skills", [])
    keywords = job_requirements.get("keywords", [])
    
    # Create prompt
    prompt = ChatPromptTemplate.from_template(COVER_LETTER_PROMPT)
    
    chain = prompt | llm
    
    # Generate cover letter
    response = chain.invoke({
        "job_description": job_description,
        "modified_resume": modified_resume,
        "must_have_skills": ", ".join(must_have[:5]),
        "nice_to_have_skills": ", ".join(nice_to_have[:3]),
        "key_phrases": ", ".join(keywords[:5])
    })
    
    cover_letter = response.content.strip()
    
    # Calculate word count
    word_count = len(cover_letter.split())
    
    print(f"✓ Cover letter generated ({word_count} words)")
    
    return {
        "cover_letter": cover_letter,
        "cover_letter_word_count": word_count
    }
```

### Step 3: Test Cover Letter Node

Create `backend/agent/nodes/test_cover_letter.py`:

```python
"""Test cover letter generation."""
import os
from dotenv import load_dotenv
from generate_cover_letter import generate_cover_letter

load_dotenv()

def test_cover_letter_generation():
    """Test basic cover letter generation."""
    
    state = {
        "job_description": """
        Senior Software Engineer at TechCorp
        
        We're looking for an experienced software engineer to join our team.
        
        Requirements:
        - 5+ years Python development
        - React and modern JavaScript
        - PostgreSQL and database design
        - RESTful API development
        - Team collaboration
        
        Nice to have:
        - AWS or cloud experience
        - Docker and Kubernetes
        - CI/CD pipeline experience
        """,
        
        "modified_resume": """
        JOHN SMITH
        Senior Software Engineer
        john.smith@email.com | (555) 123-4567
        
        EXPERIENCE
        
        Senior Developer at WebCo (2020-Present)
        - Lead team of 5 engineers building scalable web applications
        - Architected microservices using Python, FastAPI, and PostgreSQL
        - Implemented CI/CD pipelines reducing deployment time by 60%
        - Deployed applications to AWS using Docker and Kubernetes
        
        Software Engineer at StartupXYZ (2018-2020)
        - Built React-based dashboard serving 10,000+ users
        - Designed PostgreSQL schemas for efficient data retrieval
        - Collaborated with product team on feature development
        
        SKILLS
        Python, JavaScript, React, PostgreSQL, FastAPI, Django, AWS, Docker, Kubernetes, Git
        """,
        
        "job_requirements": {
            "must_have_skills": [
                "Python development",
                "React",
                "PostgreSQL",
                "RESTful API",
                "Team collaboration"
            ],
            "nice_to_have_skills": [
                "AWS",
                "Docker",
                "Kubernetes",
                "CI/CD"
            ],
            "keywords": [
                "senior software engineer",
                "scalable applications",
                "database design",
                "team leadership"
            ]
        }
    }
    
    result = generate_cover_letter(state)
    
    assert "cover_letter" in result
    assert "cover_letter_word_count" in result
    
    letter = result["cover_letter"]
    word_count = result["cover_letter_word_count"]
    
    print("\n" + "="*60)
    print("GENERATED COVER LETTER")
    print("="*60)
    print(letter)
    print("="*60)
    print(f"Word count: {word_count}")
    print("="*60)
    
    # Validation checks
    assert 250 <= word_count <= 500, f"Letter too short/long: {word_count} words"
    assert "Dear" in letter, "Missing salutation"
    assert "Sincerely" in letter or "Best regards" in letter, "Missing closing"
    assert "JOHN" in letter or "John" in letter, "Missing candidate name"
    
    print("\n✓ All assertions passed!")


if __name__ == "__main__":
    test_cover_letter_generation()
```

Run test:

```bash
cd backend/agent/nodes
python test_cover_letter.py
```

**Expected output:**
```
🎯 Generating cover letter...
✓ Cover letter generated (347 words)

============================================================
GENERATED COVER LETTER
============================================================
Dear Hiring Manager,

I am excited to apply for the Senior Software Engineer position at TechCorp...
[... personalized letter ...]

Sincerely,
John Smith
============================================================
Word count: 347
============================================================

✓ All assertions passed!
```

### Step 4: Commit Cover Letter Node

```bash
git add backend/agent/nodes/generate_cover_letter.py backend/agent/nodes/test_cover_letter.py
git commit -m "AG-51: Create cover letter generation node

- Generate personalized 300-400 word cover letters
- Use optimized resume + job requirements
- Extract company/position from JD
- Professional tone with specific examples
- Test suite with validation
- Dev 3 completes LangGraph training! 🎓"

git push origin feature/AG-51-cover-letter-node
```

**Jira:** Move AG-51 to "Code Review"

---

## 👨‍💻 Dev 2 (Sinan): AG-52 - Cover Letter Workflow Integration

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-52-cover-letter-workflow
```

**Jira:** Move AG-52 to "In Progress"

### Step 2: Update Agent State

Edit `backend/agent/state.py` - add cover letter fields:

```python
from typing import TypedDict, List, Optional

class ResumeAgentState(TypedDict, total=False):
    # ... existing fields ...
    
    # Cover letter fields (NEW)
    generate_cover_letter: bool  # Flag to request cover letter
    cover_letter: Optional[str]  # Generated cover letter
    cover_letter_word_count: Optional[int]  # Word count
```

### Step 3: Update Workflow

Edit `backend/agent/workflow.py`:

```python
"""
Resume Optimization Agent Workflow

Now with optional cover letter generation!
"""
from langgraph.graph import StateGraph, END

from .state import ResumeAgentState
from .nodes.job_requirements import extract_job_requirements
from .nodes.resume_analysis import analyze_resume
from .nodes.scoring import score_initial, score_modified
from .nodes.planning import plan_improvements
from .nodes.modification import modify_resume
from .nodes.generate_cover_letter import generate_cover_letter


def should_generate_cover_letter(state: ResumeAgentState) -> str:
    """
    Decision function for cover letter generation.
    
    Returns:
        "generate" if user requested cover letter
        "skip" otherwise
    """
    if state.get("generate_cover_letter", False):
        return "generate"
    return "skip"


def create_agent_workflow() -> StateGraph:
    """Build the LangGraph workflow with optional cover letter."""
    
    workflow = StateGraph(ResumeAgentState)
    
    # Add all nodes
    workflow.add_node("extract_requirements", extract_job_requirements)
    workflow.add_node("analyze_resume", analyze_resume)
    workflow.add_node("score_initial", score_initial)
    workflow.add_node("plan_improvements", plan_improvements)
    workflow.add_node("modify_resume", modify_resume)
    workflow.add_node("score_modified", score_modified)
    workflow.add_node("generate_cover_letter", generate_cover_letter)  # NEW
    
    # Main workflow (same as before)
    workflow.set_entry_point("extract_requirements")
    workflow.add_edge("extract_requirements", "analyze_resume")
    workflow.add_edge("analyze_resume", "score_initial")
    workflow.add_edge("score_initial", "plan_improvements")
    workflow.add_edge("plan_improvements", "modify_resume")
    workflow.add_edge("modify_resume", "score_modified")
    
    # Conditional edge: generate cover letter or finish
    workflow.add_conditional_edges(
        "score_modified",
        should_generate_cover_letter,
        {
            "generate": "generate_cover_letter",
            "skip": END
        }
    )
    
    # After cover letter, we're done
    workflow.add_edge("generate_cover_letter", END)
    
    return workflow.compile()


agent_app = create_agent_workflow()


def run_optimization(
    job_description: str,
    resume: str,
    user_id: str = "anonymous",
    run_id: str = None,
    generate_cover_letter: bool = False  # NEW parameter
) -> dict:
    """Run the full optimization workflow."""
    import uuid
    
    if run_id is None:
        run_id = str(uuid.uuid4())
    
    initial_state = {
        "run_id": run_id,
        "user_id": user_id,
        "job_description": job_description,
        "original_resume": resume,
        "generate_cover_letter": generate_cover_letter,  # Pass the flag
        "job_requirements": None,
        "resume_analysis": None,
        "ats_score_before": None,
        "ats_score_after": None,
        "improvement_delta": None,
        "improvement_plan": None,
        "modified_resume": None,
        "cover_letter": None,
        "cover_letter_word_count": None
    }
    
    result = agent_app.invoke(initial_state)
    
    return result
```

### Step 4: Test Workflow Integration

Create `backend/agent/test_cover_letter_workflow.py`:

```python
"""Test workflow with cover letter."""
import os
from dotenv import load_dotenv
from workflow import run_optimization

load_dotenv()

def test_without_cover_letter():
    """Test workflow without cover letter flag."""
    print("\n=== Test 1: Without Cover Letter ===")
    
    result = run_optimization(
        job_description="Python Developer needed. 3+ years experience.",
        resume="John Doe. Python developer with 4 years experience.",
        generate_cover_letter=False
    )
    
    assert result["modified_resume"] is not None
    assert result["ats_score_after"] is not None
    assert result["cover_letter"] is None  # Should NOT have cover letter
    
    print("✓ Workflow completed without cover letter")


def test_with_cover_letter():
    """Test workflow with cover letter flag."""
    print("\n=== Test 2: With Cover Letter ===")
    
    result = run_optimization(
        job_description="Senior Python Engineer at TechCorp. Requirements: 5+ years Python, React, PostgreSQL.",
        resume="Jane Smith. Senior Engineer with 6 years Python, React, and database experience.",
        generate_cover_letter=True  # Request cover letter
    )
    
    assert result["modified_resume"] is not None
    assert result["ats_score_after"] is not None
    assert result["cover_letter"] is not None  # Should HAVE cover letter
    assert result["cover_letter_word_count"] > 200
    
    print(f"✓ Workflow completed WITH cover letter ({result['cover_letter_word_count']} words)")
    print("\nCover Letter Preview:")
    print(result["cover_letter"][:200] + "...")


if __name__ == "__main__":
    test_without_cover_letter()
    test_with_cover_letter()
    print("\n✓ All workflow tests passed!")
```

Run test:

```bash
cd backend/agent
python test_cover_letter_workflow.py
```

### Step 5: Commit Workflow Integration

```bash
git add backend/agent/workflow.py backend/agent/state.py backend/agent/test_cover_letter_workflow.py
git commit -m "AG-52: Integrate cover letter into workflow

- Add generate_cover_letter flag to state
- Conditional edge routes to cover letter node or END
- Cover letter only generated when requested
- Updated run_optimization() function
- Integration tests pass"

git push origin feature/AG-52-cover-letter-workflow
```

**Jira:** Move AG-52 to "Code Review"

---

## 👨‍💻 Dev 1 (Shabas): AG-53 - Cover Letter API Endpoint

### Step 1: Create Branch

```bash
git checkout shabas-Dev && git pull origin main
git checkout -b feature/AG-53-cover-letter-api
```

**Jira:** Move AG-53 to "In Progress"

### Step 2: Add Database Column

Create migration script `backend/migrations/add_cover_letter_column.sql`:

```sql
-- Add cover_letter column to agent_runs table
ALTER TABLE agent_runs 
ADD COLUMN IF NOT EXISTS cover_letter TEXT;

-- Add word count for analytics
ALTER TABLE agent_runs
ADD COLUMN IF NOT EXISTS cover_letter_word_count INTEGER;
```

Run migration in Supabase SQL Editor:

1. Go to https://supabase.com → Your Project → SQL Editor
2. Paste the SQL above
3. Click "Run"
4. Verify: Go to Table Editor → agent_runs → See new columns

### Step 3: Update Database Model

Edit `backend/models/agent_run.py`:

```python
from sqlalchemy import Column, Integer, Float, String, Text, DateTime, ForeignKey, JSON
from sqlalchemy.dialects.postgresql import UUID
from database.connection import Base
import uuid
from datetime import datetime

class AgentRun(Base):
    __tablename__ = "agent_runs"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    user_id = Column(UUID(as_uuid=True), ForeignKey("users.id"), nullable=True)
    
    # Input
    job_description = Column(Text, nullable=False)
    original_resume = Column(Text, nullable=False)
    
    # Output
    modified_resume = Column(Text)
    cover_letter = Column(Text)  # NEW
    cover_letter_word_count = Column(Integer)  # NEW
    
    # Scores
    ats_score_before = Column(Float)
    ats_score_after = Column(Float)
    improvement_delta = Column(Float)
    
    # Analysis
    job_requirements = Column(JSON)
    resume_analysis = Column(JSON)
    improvement_plan = Column(JSON)
    
    # Metadata
    iteration_count = Column(Integer, default=0)
    status = Column(String(20), default="pending")
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
    
    def to_dict(self):
        """Convert to dict for API response."""
        return {
            "id": str(self.id),
            "user_id": str(self.user_id) if self.user_id else None,
            "job_description": self.job_description,
            "original_resume": self.original_resume,
            "modified_resume": self.modified_resume,
            "cover_letter": self.cover_letter,  # Include cover letter
            "cover_letter_word_count": self.cover_letter_word_count,
            "ats_score_before": self.ats_score_before,
            "ats_score_after": self.ats_score_after,
            "improvement_delta": self.improvement_delta,
            "job_requirements": self.job_requirements,
            "resume_analysis": self.resume_analysis,
            "improvement_plan": self.improvement_plan,
            "iteration_count": self.iteration_count,
            "status": self.status,
            "created_at": self.created_at.isoformat() if self.created_at else None,
            "completed_at": self.completed_at.isoformat() if self.completed_at else None
        }
```

### Step 4: Update API Endpoint

Edit `backend/routers/agent.py`:

```python
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

from models.agent_run import AgentRun
from database.connection import get_db
from agent.workflow import run_optimization
from auth.dependencies import get_current_user_id

router = APIRouter(prefix="/api/agent", tags=["agent"])


class OptimizationRequest(BaseModel):
    resume_text: str = Field(..., min_length=100)
    job_description: str = Field(..., min_length=50)
    generate_cover_letter: bool = False  # NEW optional parameter


class OptimizationResponse(BaseModel):
    id: str
    original_resume: str
    modified_resume: str
    cover_letter: Optional[str] = None  # NEW
    cover_letter_word_count: Optional[int] = None  # NEW
    ats_score_before: float
    ats_score_after: float
    improvement_delta: float
    job_requirements: dict
    resume_analysis: dict
    improvement_plan: dict
    created_at: str


@router.post("/optimize", response_model=OptimizationResponse)
def optimize_resume(
    request: OptimizationRequest,
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """
    Optimize resume for job description.
    Optionally generate cover letter.
    """
    # Run agent workflow
    result = run_optimization(
        job_description=request.job_description,
        resume=request.resume_text,
        user_id=user_id,
        generate_cover_letter=request.generate_cover_letter  # Pass flag
    )
    
    # Save to database
    run = AgentRun(
        user_id=user_id,
        job_description=request.job_description,
        original_resume=request.resume_text,
        modified_resume=result["modified_resume"],
        cover_letter=result.get("cover_letter"),  # NEW
        cover_letter_word_count=result.get("cover_letter_word_count"),  # NEW
        ats_score_before=result["ats_score_before"],
        ats_score_after=result["ats_score_after"],
        improvement_delta=result["improvement_delta"],
        job_requirements=result["job_requirements"],
        resume_analysis=result["resume_analysis"],
        improvement_plan=result["improvement_plan"],
        iteration_count=result.get("iteration_count", 0),
        status="completed",
        completed_at=datetime.utcnow()
    )
    
    db.add(run)
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
        created_at=run.created_at.isoformat()
    )


@router.get("/runs/{run_id}", response_model=OptimizationResponse)
def get_optimization_run(
    run_id: str,
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """Get specific optimization run."""
    run = db.query(AgentRun).filter(
        AgentRun.id == run_id,
        AgentRun.user_id == user_id
    ).first()
    
    if not run:
        raise HTTPException(status_code=404, detail="Run not found")
    
    return OptimizationResponse(**run.to_dict())


@router.get("/runs")
def list_optimization_runs(
    user_id: str = Depends(get_current_user_id),
    db = Depends(get_db)
):
    """Get all runs for current user."""
    runs = db.query(AgentRun).filter(
        AgentRun.user_id == user_id
    ).order_by(AgentRun.created_at.desc()).limit(50).all()
    
    return [run.to_dict() for run in runs]
```

### Step 5: Test API

```bash
cd backend
uvicorn main:app --reload
```

Test with cover letter:

```bash
# Login first
TOKEN=$(curl -s -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"Test1234"}' | jq -r '.access_token')

# Optimize WITH cover letter
curl -X POST http://localhost:8000/api/agent/optimize \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "resume_text": "John Smith. Senior Engineer with 6 years Python and React experience. Built scalable APIs. PostgreSQL expert.",
    "job_description": "Senior Software Engineer at TechCorp. 5+ years Python, React, PostgreSQL required. Team leadership experience preferred.",
    "generate_cover_letter": true
  }' | jq '.'
```

**Expected response includes:**
```json
{
  "id": "...",
  "modified_resume": "...",
  "cover_letter": "Dear Hiring Manager,\n\nI am excited to apply for...",
  "cover_letter_word_count": 342,
  "ats_score_before": 68.5,
  "ats_score_after": 85.2,
  ...
}
```

### Step 6: Commit API Changes

```bash
git add backend/routers/agent.py backend/models/agent_run.py backend/migrations/
git commit -m "AG-53: Add cover letter to API endpoint

- Add generate_cover_letter optional parameter
- Store cover letter in database
- Include cover letter in API response
- Database migration for new columns
- API tests pass with/without cover letter"

git push origin feature/AG-53-cover-letter-api
```

**Jira:** Move AG-53 to "Code Review"

---

## 📞 2:00 PM - Review Calls

### Marva → Shabas: Review AG-51

**Marva:** "Shabas, I've created the cover letter generation node. Can you review?"

**Shabas:** *Reviews generate_cover_letter.py and tests*

"Excellent work! The prompt is detailed and the output quality is great. The 300-400 word target is perfect. Approved! ✅  
Also, congrats on completing all 3 node types - you're now a LangGraph expert! 🎓"

**Jira:** AG-51 moved to "Done"

---

### Sinan → Marva: Review AG-52

**Sinan:** "Marva, I've integrated your cover letter node with conditional routing. Please review!"

**Marva:** *Reviews workflow.py changes and integration tests*

"Perfect integration! The conditional edge logic is clean and the flag properly controls whether to generate the letter. Tests confirm both paths work. Approved! ✅"

**Jira:** AG-52 moved to "Done"

---

### Shabas → Sinan: Review AG-53

**Shabas:** "Sinan, I've updated the API to support cover letters. Can you review?"

**Sinan:** *Reviews agent.py router, database migration, and model changes*

"Great work! The optional parameter is elegant, database migration is clean, and the API response properly includes the cover letter. Tested with curl and it works perfectly. Approved! ✅"

**Jira:** AG-53 moved to "Done"

---

## 🔀 3:00 PM - Merge to Main

### Merge All Cover Letter Features

```bash
# Merge AG-51 (Cover Letter Node)
git checkout main
git pull origin main
git merge feature/AG-51-cover-letter-node
git push origin main

# Merge AG-52 (Workflow Integration)
git merge feature/AG-52-cover-letter-workflow
git push origin main

# Merge AG-53 (API Endpoint)
git merge feature/AG-53-cover-letter-api
git push origin main

# Clean up branches
git branch -d feature/AG-51-cover-letter-node
git branch -d feature/AG-52-cover-letter-workflow
git branch -d feature/AG-53-cover-letter-api
```

---

## ✅ 3:30 PM - End-to-End Verification

### Test Complete Flow

```bash
# 1. Start backend
cd backend
uvicorn main:app --reload

# 2. Test WITHOUT cover letter
curl -X POST http://localhost:8000/api/agent/optimize \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "resume_text": "Developer with 3 years experience",
    "job_description": "Looking for developer with experience",
    "generate_cover_letter": false
  }' | jq '.cover_letter'
# Should be: null

# 3. Test WITH cover letter
curl -X POST http://localhost:8000/api/agent/optimize \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "resume_text": "John Smith. Senior Engineer with 6 years Python, React, PostgreSQL. Team lead at WebCo.",
    "job_description": "Senior Software Engineer at TechCorp. 5+ years Python, React required. Team leadership preferred.",
    "generate_cover_letter": true
  }' | jq '.cover_letter'
# Should return 300-400 word cover letter
```

### Verification Checklist

| # | Test | Expected Result | ✓ |
|---|------|-----------------|---|
| 1 | API call without flag | No cover letter in response | |
| 2 | API call with flag=false | No cover letter in response | |
| 3 | API call with flag=true | Cover letter generated | |
| 4 | Cover letter length | 300-400 words | |
| 5 | Cover letter quality | Professional tone, specific examples | |
| 6 | Database storage | Cover letter saved in agent_runs | |
| 7 | API response includes word count | cover_letter_word_count field present | |
| 8 | Workflow without cover letter | Faster execution ~30-40 sec | |
| 9 | Workflow with cover letter | Slower execution ~45-60 sec | |
| 10 | Cover letter mentions candidate name | Extracted from resume | |

---

## 📁 Updated Folder Structure

```
Resume-Agent/
└── backend/
    ├── agent/
    │   ├── nodes/
    │   │   ├── job_requirements.py
    │   │   ├── resume_analysis.py
    │   │   ├── scoring.py
    │   │   ├── planning.py
    │   │   ├── modification.py
    │   │   └── generate_cover_letter.py  ✓ NEW
    │   ├── state.py                       ✓ Updated
    │   ├── workflow.py                    ✓ Updated
    │   └── test_cover_letter_workflow.py  ✓ NEW
    ├── models/
    │   └── agent_run.py                   ✓ Updated (cover_letter fields)
    ├── routers/
    │   └── agent.py                       ✓ Updated (optional param)
    └── migrations/
        └── add_cover_letter_column.sql    ✓ NEW
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-51: Cover Letter Generation Node | Dev 3 (Marva) | 5 | ✅ Done |
| AG-52: Workflow Integration | Dev 2 (Sinan) | 2 | ✅ Done |
| AG-53: API Endpoint | Dev 1 (Shabas) | 3 | ✅ Done |

**Total Story Points Completed:** 10  
**Dev 1 (Shabas):** 3 points | **Dev 2 (Sinan):** 2 points | **Dev 3 (Marva):** 5 points

---

## 🎉 Cover Letter Feature Complete!

Day 13 achievements:
- ✅ Cover letter generation node with GPT-4o-mini
- ✅ Conditional workflow routing (generate or skip)
- ✅ API support for optional cover letter
- ✅ Database storage for cover letters
- ✅ 300-400 word professional letters
- ✅ Dev 3 completes LangGraph training! 🎓

**All 3 developers now trained on LangGraph nodes!**

**Tomorrow: Cover Letter UI & History Page! 📜**

---

**← [Day 12](./day-12.md)** | **[Day 14: Cover Letter UI →](./day-14.md)**
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
git commit -m "AG-21: Add conditional edges for iteration

- Create decision node to check score vs target
- Add conditional edge to loop back or finish
- Log each iteration decision
- Default max 3 iterations

Co-authored-by: Sinan <sinan@example.com>"

git push origin feature/AG-21-conditional-edges
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
