# Day 3: Add More Nodes

> **Date:** Sprint 1, Day 3  
> **Focus:** Build 3 more LangGraph nodes (all devs get hands-on)  
> **Vertical Slice:** 3-node workflow runs: Extract → Analyze → Score

---

## 🎯 Today's Goal

By end of today:
- ✅ Resume Analysis node working (Dev 2 builds it)
- ✅ ATS Scoring node working (Dev 3 builds it - LangGraph experience!)
- ✅ 3-node workflow runs end-to-end (Dev 1 integrates)
- ✅ All 3 developers have built a LangGraph node

---

## 📋 Morning: Add These Tasks to Jira Backlog

---

### Task AG-18: Resume Analysis Node

| Field | Value |
|-------|-------|
| **Task ID** | AG-18 |
| **Title** | Resume Analysis Node |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Build a LangGraph node that analyzes resumecontent using GPT-4o-mini. This node identifies strengths, weaknesses, missing keywords, and areas for improvement. Dev 2 gets full LangGraph node building experience!

**What You'll Learn:**
- Creating LangGraph nodes independently
- Crafting prompts for resume analysis
- Structuring LLM responses
- Updating agent state

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/resume_analysis.py`
- [ ] Node analyzes resume for: strengths, weaknesses, missing_items, suggestions
- [ ] Returns dict that updates state's `resume_analysis` field
- [ ] Includes test with sample resume
- [ ] PR merged to main

**Dependencies:** AG-15 (Job requirements node exists as reference)

**Related Tasks:** Similar pattern to AG-15 but different analysis

---

### Task AG-19: ATS Scoring Function

| Field | Value |
|-------|-------|
| **Task ID** | AG-19 |
| **Title** | ATS Scoring Function |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Build a LangGraph node that calculates ATS (Applicant Tracking System) scores for resumes. This is Dev 3's FIRST LangGraph node - now everyone has hands-on experience! Uses GPT-4o-mini to score resume match quality.

**What You'll Learn:**
- Creating LangGraph nodes
- Using LLMs for scoring/evaluation
- Returning scored data in state updates
- LangGraph node patterns

**Acceptance Criteria:**
- [ ] File created: `backend/agent/nodes/scoring.py`
- [ ] Node scores resume against job requirements
- [ ] Returns score 0-100 with breakdown
- [ ] Updates state's `ats_score_before` field
- [ ] Includes test file
- [ ] PR merged to main

**Dependencies:** AG-15 (needs job requirements), AG-18 (needs resume analysis)

**Related Tasks:** Dev 3 now has LangGraph experience!

---

### Task AG-20: 3-Node Workflow Integration

| Field | Value |
|-------|-------|
| **Task ID** | AG-20 |
| **Title** | 3-Node Workflow Integration |
| **Type** | Task |
| **Epic** | LangGraph Agent Core |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Connect the 3 LangGraph nodes built (Job Requirements from yesterday, Resume Analysis, and Scoring) into a working workflow. This creates the first multi-node agent pipeline that runs end-to-end!

**What You'll Learn:**
- Connecting multiple LangGraph nodes
- Designing node execution order
- State flow across nodes
- Testing multi-node workflows

**Acceptance Criteria:**
- [ ] File created: `backend/agent/three_node_workflow.py`
- [ ] Workflow runs: extract requirements → analyze resume → score resume
- [ ] State flows correctly between nodes
- [ ] Test script shows all 3 nodes executing
- [ ] Prints final score and analysis
- [ ] PR merged to main

**Dependencies:** AG-15, AG-18, AG-19 (all 3 nodes must exist)

**Related Tasks:** First real multi-step agent workflow!

---

## 🕘 9:00 AM - Daily Standup

**Expected Updates:**
- **Dev 1:** Yesterday: Agent state schema. Today: AG-15 Job Requirements node (pair with Dev 2).
- **Dev 2:** Yesterday: FastAPI structure. Today: AG-15 (pair) then AG-17 Scoring.
- **Dev 3:** Yesterday: React setup. Today: AG-28 Login/Register UI.

---

## 👥 Dev 1 + Dev 2 (PAIR): AG-15 - Job Requirements Node

### About Pair Programming

**How it works:**
- **Driver:** Types the code
- **Navigator:** Reviews, thinks ahead, catches errors
- **Switch every 45 minutes**
- Communication is key - talk through your thinking

**Setup:**
- Sit side by side (or share screen if remote)
- Use one computer
- Dev 1 drives first

---

### What to Learn First (30 minutes - together)

**Read together:**
- LangChain Chat Models: https://python.langchain.com/docs/modules/model_io/chat/

**Key Concepts:**
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# Create LLM
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# Create prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{input}")
])

# Create chain
chain = prompt | llm

# Invoke
result = chain.invoke({"input": "Hello!"})
print(result.content)
```

---

### Step-by-Step Instructions

#### Step 1: Setup (Dev 1 drives first)

```bash
cd resume-agent
git checkout dev-shabas
git pull origin main
git checkout -b feature/AG-15-job-requirements-node
```

**Jira Update:** Move AG-15 to "In Progress"

#### Step 2: Create the Node File

Create file: `backend/agent/nodes/job_requirements.py`

```python
"""
Job Requirements Extraction Node

This node extracts structured requirements from a job description.
It uses an LLM to parse the JD and identify:
- Must-have skills
- Nice-to-have skills  
- Keywords for ATS
- Experience requirements
- Other important details
"""
import json
import os
from typing import Dict

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

from ..state import ResumeAgentState

# The prompt we send to the LLM
EXTRACTION_PROMPT = """You are an expert at analyzing job descriptions.

Extract the following from this job description:

1. **must_have_skills**: Skills explicitly required (e.g., "3+ years Python required")
2. **nice_to_have_skills**: Skills that are preferred but not required
3. **keywords**: ATS keywords to include in resume
4. **years_experience**: Required years of experience (number or "not specified")
5. **education**: Education requirements
6. **responsibilities**: Main job responsibilities
7. **company_values**: Any mentioned company values or culture points

Job Description:
---
{job_description}
---

Return your response as valid JSON only. No explanation, just the JSON object.

Example format:
{{
    "must_have_skills": ["Python", "Django", "PostgreSQL"],
    "nice_to_have_skills": ["Docker", "AWS"],
    "keywords": ["backend", "API", "scalable", "agile"],
    "years_experience": 3,
    "education": "Bachelor's in Computer Science",
    "responsibilities": ["Build APIs", "Write tests"],
    "company_values": ["Innovation", "Teamwork"]
}}
"""


def extract_job_requirements(state: ResumeAgentState) -> Dict:
    """
    LangGraph Node: Extract requirements from job description.
    
    Args:
        state: The current agent state
        
    Returns:
        Dict with job_requirements field to merge into state
    """
    # Create LLM client
    llm = ChatOpenAI(
        model="gpt-4o-mini",  # Fast and cheap for extraction
        temperature=0,        # Deterministic output
        api_key=os.getenv("OPENAI_API_KEY")
    )
    
    # Create prompt template
    prompt = ChatPromptTemplate.from_messages([
        ("system", "You extract structured data from job descriptions. Return only valid JSON."),
        ("user", EXTRACTION_PROMPT)
    ])
    
    # Create chain: prompt -> LLM
    chain = prompt | llm
    
    # Run extraction
    result = chain.invoke({
        "job_description": state["job_description"]
    })
    
    # Parse the JSON response
    try:
        requirements = json.loads(result.content)
    except json.JSONDecodeError:
        # Fallback if LLM returns invalid JSON
        requirements = {
            "must_have_skills": [],
            "nice_to_have_skills": [],
            "keywords": [],
            "years_experience": "not specified",
            "education": "not specified",
            "responsibilities": [],
            "company_values": [],
            "parse_error": True
        }
    
    # Log the decision (this is what makes it "agentic")
    decision = {
        "node": "extract_job_requirements",
        "action": "extracted_requirements",
        "skills_found": len(requirements.get("must_have_skills", [])),
        "keywords_found": len(requirements.get("keywords", []))
    }
    
    # Return update to merge into state
    return {
        "job_requirements": requirements,
        "decision_log": state.get("decision_log", []) + [decision]
    }
```

**⏰ 45 min mark - SWITCH DRIVERS! Dev 2 now types.**

#### Step 3: Create Test File

Create file: `backend/agent/nodes/test_job_requirements.py`

```python
"""
Tests for Job Requirements Extraction Node

Run with: python test_job_requirements.py
Note: Requires OPENAI_API_KEY environment variable
"""
import os
from dotenv import load_dotenv

# Load env variables
load_dotenv()

# Check API key
if not os.getenv("OPENAI_API_KEY"):
    print("❌ Error: OPENAI_API_KEY not set")
    print("Set it in backend/.env file")
    exit(1)

from job_requirements import extract_job_requirements


def test_basic_extraction():
    """Test extraction from a simple job description."""
    
    # Sample job description
    job_description = """
    Software Engineer - Backend
    
    We're looking for a talented Backend Engineer to join our team.
    
    Requirements:
    - 3+ years of Python experience
    - Strong knowledge of Django or FastAPI
    - Experience with PostgreSQL
    - Familiarity with Docker and Kubernetes
    
    Nice to have:
    - AWS experience
    - Experience with microservices
    - CI/CD pipeline experience
    
    Responsibilities:
    - Build and maintain REST APIs
    - Write comprehensive tests
    - Participate in code reviews
    - Collaborate with frontend team
    
    About us:
    We value innovation, collaboration, and continuous learning.
    """
    
    # Create minimal state
    state = {
        "run_id": "test-run",
        "user_id": "test-user",
        "job_description": job_description,
        "original_resume": "",
        "decision_log": []
    }
    
    # Run extraction
    print("Running job requirements extraction...")
    result = extract_job_requirements(state)
    
    # Verify result
    assert "job_requirements" in result, "Should return job_requirements"
    
    requirements = result["job_requirements"]
    
    print("\n=== Extracted Requirements ===")
    print(f"Must-have skills: {requirements.get('must_have_skills', [])}")
    print(f"Nice-to-have skills: {requirements.get('nice_to_have_skills', [])}")
    print(f"Keywords: {requirements.get('keywords', [])}")
    print(f"Years experience: {requirements.get('years_experience')}")
    
    # Basic assertions
    must_have = requirements.get("must_have_skills", [])
    assert len(must_have) > 0, "Should find at least one must-have skill"
    
    # Check for expected skills (case-insensitive)
    must_have_lower = [s.lower() for s in must_have]
    assert any("python" in s for s in must_have_lower), "Should find Python"
    
    print("\n✓ Basic extraction test passed!")
    

def test_decision_log():
    """Test that decision log is updated."""
    
    state = {
        "run_id": "test-run-2",
        "user_id": "test-user",
        "job_description": "Looking for a JavaScript developer with React experience.",
        "original_resume": "",
        "decision_log": []
    }
    
    result = extract_job_requirements(state)
    
    assert "decision_log" in result, "Should return decision_log"
    assert len(result["decision_log"]) > 0, "Should have at least one decision"
    
    decision = result["decision_log"][0]
    assert decision["node"] == "extract_job_requirements"
    
    print("✓ Decision log test passed!")


if __name__ == "__main__":
    test_basic_extraction()
    test_decision_log()
    print("\n🎉 All job requirements tests passed!")
```

#### Step 4: Test the Node

```bash
cd backend

# Make sure .env has OPENAI_API_KEY
cat .env | grep OPENAI  # Should show your key (or be set)

# Run tests
cd agent/nodes
python test_job_requirements.py
```

**Expected Output:**
```
Running job requirements extraction...

=== Extracted Requirements ===
Must-have skills: ['Python', 'Django', 'FastAPI', 'PostgreSQL', 'Docker', 'Kubernetes']
Nice-to-have skills: ['AWS', 'microservices', 'CI/CD']
Keywords: ['Backend', 'APIs', 'scalable', ...]
Years experience: 3

✓ Basic extraction test passed!
✓ Decision log test passed!

🎉 All job requirements tests passed!
```

#### Step 5: Commit

```bash
cd ../../..  # Back to resume-agent root

git add backend/agent/nodes/

git commit -m "AG-15: Create job requirements extraction node

- Add LLM-powered extraction using gpt-4o-mini
- Extract must-have skills, nice-to-have skills, keywords
- Add JSON parsing with fallback
- Include decision logging
- Add test file with example JD

Co-authored-by: Sinan <sinan@example.com>"

git push origin feature/AG-15-job-requirements-node
```

**Jira Update:** Move AG-15 to "In Review"

#### Step 6: Open PR, Get Review, Merge

PR Title: `AG-15: Create job requirements extraction node`

After merge:
```bash
git checkout dev-shabas
git pull origin main
git branch -d feature/AG-15-job-requirements-node
```

**Jira Update:** Move AG-15 to "Done"

---

## 👨‍💻 Dev 2 (Sinan): AG-17 - ATS Scoring Function

*Start this after pair programming session ends (around 2 PM)*

### What to Learn First (15 minutes)

**Understand ATS Scoring:**
- ATS = Applicant Tracking System
- Companies use ATS to filter resumes
- Score based on keyword matches, format, sections

**Our Scoring Formula:**
- Keywords match: 40%
- Skills match: 30%
- Format quality: 15%
- Section presence: 15%

---

### Step-by-Step Instructions

#### Step 1: Create Feature Branch

```bash
git checkout sinan-Dev
git pull origin main
git checkout -b feature/AG-17-ats-scoring
```

**Jira Update:** Move AG-17 to "In Progress"

#### Step 2: Create Scoring Module

Create file: `backend/agent/scoring.py`

```python
"""
ATS Scoring Module

Calculates an ATS (Applicant Tracking System) score for a resume
based on how well it matches job requirements.

Scoring breakdown:
- Keywords: 40 points (weighted by importance)
- Must-have Skills: 30 points
- Format Quality: 15 points
- Section Presence: 15 points

Total: 0-100 points
"""
from typing import Dict, List, Tuple


def calculate_ats_score(
    resume: str,
    requirements: Dict
) -> Dict:
    """
    Calculate ATS score for a resume against job requirements.
    
    Args:
        resume: Full text of the resume
        requirements: Dict from job requirements extraction
        
    Returns:
        Dict with score and breakdown
    """
    resume_lower = resume.lower()
    
    # Initialize scores
    keyword_score = 0.0
    skills_score = 0.0
    format_score = 0.0
    section_score = 0.0
    
    matched_keywords = []
    matched_skills = []
    missing_skills = []
    
    # === KEYWORD MATCHING (40 points) ===
    keywords = requirements.get("keywords", [])
    if keywords:
        for keyword in keywords:
            if keyword.lower() in resume_lower:
                matched_keywords.append(keyword)
        
        if len(keywords) > 0:
            keyword_score = (len(matched_keywords) / len(keywords)) * 40
    
    # === SKILLS MATCHING (30 points) ===
    must_have_skills = requirements.get("must_have_skills", [])
    if must_have_skills:
        for skill in must_have_skills:
            if skill.lower() in resume_lower:
                matched_skills.append(skill)
            else:
                missing_skills.append(skill)
        
        if len(must_have_skills) > 0:
            skills_score = (len(matched_skills) / len(must_have_skills)) * 30
    
    # === FORMAT QUALITY (15 points) ===
    # Check resume length (too short or too long is bad)
    word_count = len(resume.split())
    
    if 400 <= word_count <= 800:
        format_score += 8  # Ideal length
    elif 300 <= word_count <= 1000:
        format_score += 5  # Acceptable
    elif 200 <= word_count <= 1200:
        format_score += 2  # Not great
    # else: 0 points (too short or too long)
    
    # Check for bullet points or lists
    if "•" in resume or "- " in resume or "* " in resume:
        format_score += 4
    
    # Check for consistent formatting (no huge blocks of text)
    paragraphs = resume.split("\n\n")
    avg_paragraph_length = sum(len(p.split()) for p in paragraphs) / max(len(paragraphs), 1)
    if avg_paragraph_length < 100:  # Shorter paragraphs = better formatting
        format_score += 3
    
    # === SECTION PRESENCE (15 points) ===
    important_sections = [
        ("experience", ["experience", "work history", "employment"]),
        ("education", ["education", "academic", "degree"]),
        ("skills", ["skills", "technical skills", "competencies"]),
        ("summary", ["summary", "objective", "profile"])
    ]
    
    sections_found = []
    for section_name, keywords_list in important_sections:
        for keyword in keywords_list:
            if keyword.lower() in resume_lower:
                sections_found.append(section_name)
                break
    
    section_score = (len(sections_found) / len(important_sections)) * 15
    
    # === CALCULATE TOTAL ===
    total_score = keyword_score + skills_score + format_score + section_score
    total_score = round(min(total_score, 100), 1)  # Cap at 100
    
    return {
        "score": total_score,
        "breakdown": {
            "keywords": round(keyword_score, 1),
            "skills": round(skills_score, 1),
            "format": round(format_score, 1),
            "sections": round(section_score, 1)
        },
        "details": {
            "matched_keywords": matched_keywords,
            "matched_skills": matched_skills,
            "missing_skills": missing_skills,
            "word_count": word_count,
            "sections_found": sections_found
        }
    }


def get_score_feedback(score_result: Dict) -> str:
    """
    Generate human-readable feedback based on score.
    
    Args:
        score_result: Result from calculate_ats_score
        
    Returns:
        String with actionable feedback
    """
    score = score_result["score"]
    details = score_result["details"]
    
    feedback_parts = []
    
    # Overall assessment
    if score >= 80:
        feedback_parts.append("Excellent match! Your resume is well-optimized for this role.")
    elif score >= 60:
        feedback_parts.append("Good match. A few improvements could make it even stronger.")
    elif score >= 40:
        feedback_parts.append("Moderate match. Consider adding more relevant keywords and skills.")
    else:
        feedback_parts.append("Low match. Significant improvements needed for this role.")
    
    # Missing skills feedback
    missing = details.get("missing_skills", [])
    if missing:
        feedback_parts.append(f"\nMissing key skills: {', '.join(missing[:5])}")
        if len(missing) > 5:
            feedback_parts.append(f"...and {len(missing) - 5} more.")
    
    # Word count feedback
    word_count = details.get("word_count", 0)
    if word_count < 300:
        feedback_parts.append("\nResume is too short. Add more detail about your experience.")
    elif word_count > 1000:
        feedback_parts.append("\nResume is too long. Focus on relevant experience.")
    
    return "\n".join(feedback_parts)
```

#### Step 3: Create Test File

Create file: `backend/agent/test_scoring.py`

```python
"""
Tests for ATS Scoring Module

Run with: python test_scoring.py
"""
from scoring import calculate_ats_score, get_score_feedback


def test_high_match_resume():
    """Test scoring for a well-matched resume."""
    
    resume = """
    John Doe
    Senior Software Engineer
    
    Summary
    Experienced Python developer with 5 years building scalable backend systems.
    
    Skills
    - Python, Django, FastAPI
    - PostgreSQL, Redis
    - Docker, Kubernetes
    - AWS, GCP
    
    Experience
    
    Senior Backend Engineer | TechCorp | 2020-Present
    • Built REST APIs serving 10M users
    • Implemented CI/CD pipelines
    • Led migration to microservices architecture
    
    Software Engineer | StartupXYZ | 2018-2020
    • Developed Python backend services
    • Wrote comprehensive unit tests
    • Collaborated with frontend team
    
    Education
    BS Computer Science, MIT, 2018
    """
    
    requirements = {
        "must_have_skills": ["Python", "Django", "PostgreSQL"],
        "nice_to_have_skills": ["Docker", "AWS"],
        "keywords": ["backend", "API", "scalable", "microservices"]
    }
    
    result = calculate_ats_score(resume, requirements)
    
    print(f"Score: {result['score']}/100")
    print(f"Breakdown: {result['breakdown']}")
    print(f"Matched skills: {result['details']['matched_skills']}")
    print(f"Missing skills: {result['details']['missing_skills']}")
    
    assert result["score"] >= 70, f"Well-matched resume should score 70+, got {result['score']}"
    assert len(result["details"]["matched_skills"]) >= 2, "Should match most skills"
    
    print("✓ High match resume test passed!")


def test_low_match_resume():
    """Test scoring for a poorly matched resume."""
    
    resume = """
    Jane Smith
    Marketing Manager
    
    Experience in digital marketing and social media management.
    Led campaigns for Fortune 500 companies.
    """
    
    requirements = {
        "must_have_skills": ["Python", "Django", "PostgreSQL"],
        "keywords": ["backend", "API", "microservices"]
    }
    
    result = calculate_ats_score(resume, requirements)
    
    print(f"Score: {result['score']}/100")
    
    assert result["score"] < 40, f"Mismatched resume should score <40, got {result['score']}"
    
    print("✓ Low match resume test passed!")


def test_feedback_generation():
    """Test that feedback is generated correctly."""
    
    result = {
        "score": 45,
        "breakdown": {"keywords": 15, "skills": 10, "format": 10, "sections": 10},
        "details": {
            "missing_skills": ["Docker", "Kubernetes"],
            "word_count": 250
        }
    }
    
    feedback = get_score_feedback(result)
    
    assert "Moderate match" in feedback, "Should mention moderate match"
    assert "Docker" in feedback, "Should mention missing skills"
    assert "too short" in feedback, "Should mention short resume"
    
    print("✓ Feedback generation test passed!")


def test_empty_requirements():
    """Test handling of empty requirements."""
    
    resume = "Some resume content here with Python and Django skills."
    requirements = {}
    
    result = calculate_ats_score(resume, requirements)
    
    assert "score" in result, "Should still return a score"
    assert result["score"] >= 0, "Score should be non-negative"
    
    print("✓ Empty requirements test passed!")


if __name__ == "__main__":
    test_high_match_resume()
    print()
    test_low_match_resume()
    print()
    test_feedback_generation()
    print()
    test_empty_requirements()
    print("\n🎉 All scoring tests passed!")
```

#### Step 4: Run Tests

```bash
cd backend/agent
python test_scoring.py
```

**Expected Output:**
```
Score: 85.0/100
Breakdown: {'keywords': 40.0, 'skills': 30.0, 'format': 15.0, 'sections': 0.0}
Matched skills: ['Python', 'Django', 'PostgreSQL']
Missing skills: []
✓ High match resume test passed!

Score: 12.5/100
✓ Low match resume test passed!

✓ Feedback generation test passed!

✓ Empty requirements test passed!

🎉 All scoring tests passed!
```

#### Step 5: Commit and PR

```bash
cd ../..  # Back to resume-agent root

git add backend/agent/

git commit -m "AG-17: Create ATS scoring function

- Implement keyword matching (40 pts)
- Implement skills matching (30 pts)
- Implement format quality checks (15 pts)
- Implement section presence checks (15 pts)
- Add feedback generation
- Add comprehensive tests"

git push origin feature/AG-17-ats-scoring
```

Open PR, merge, cleanup.

**Jira Update:** Move AG-17 to "Done"

---

## 👨‍💻 Dev 3 (Marva): AG-28 - Login & Register UI

### What to Learn First (20 minutes)

**Read:**
- React Forms: https://react.dev/reference/react-dom/components/input
- useState Hook: https://react.dev/reference/react/useState

**Key Patterns:**
```jsx
const [email, setEmail] = useState('');

<input 
  type="email" 
  value={email} 
  onChange={(e) => setEmail(e.target.value)} 
/>
```

---

### Step-by-Step Instructions

#### Step 1: Create Feature Branch

```bash
cd resume-agent
git checkout dev-marva
git pull origin main
git checkout -b feature/AG-28-auth-ui
```

**Jira Update:** Move AG-28 to "In Progress"

#### Step 2: Create Pages Directory

```bash
mkdir -p frontend/src/pages
```

#### Step 3: Create Login Page

Create file: `frontend/src/pages/Login.jsx`

```jsx
/**
 * Login Page Component
 * 
 * Handles user login with email and password.
 * Validates input before submission and shows appropriate errors.
 */
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import './Auth.css';

function Login() {
  // Form state
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  // UI state
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  const navigate = useNavigate();

  /**
   * Validate form inputs
   */
  const validateForm = () => {
    setError('');
    
    if (!email.trim()) {
      setError('Email is required');
      return false;
    }
    
    // Basic email format check
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Please enter a valid email address');
      return false;
    }
    
    if (!password) {
      setError('Password is required');
      return false;
    }
    
    if (password.length < 6) {
      setError('Password must be at least 6 characters');
      return false;
    }
    
    return true;
  };

  /**
   * Handle form submission
   */
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      // API call will be added later
      console.log('Login attempt:', { email });
      
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // For now, just navigate to dashboard
      // In real implementation, this will call the backend
      navigate('/dashboard');
      
    } catch (err) {
      setError('Login failed. Please check your credentials.');
      console.error('Login error:', err);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>Welcome Back</h1>
        <p className="auth-subtitle">Sign in to continue to Resume Agent</p>
        
        {error && (
          <div className="error-message">
            {error}
          </div>
        )}
        
        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              disabled={loading}
              autoComplete="email"
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Enter your password"
              disabled={loading}
              autoComplete="current-password"
            />
          </div>
          
          <button 
            type="submit" 
            className="auth-button"
            disabled={loading}
          >
            {loading ? 'Signing in...' : 'Sign In'}
          </button>
        </form>
        
        <p className="auth-footer">
          Don't have an account?{' '}
          <Link to="/register">Create one</Link>
        </p>
      </div>
    </div>
  );
}

export default Login;
```

#### Step 4: Create Register Page

Create file: `frontend/src/pages/Register.jsx`

```jsx
/**
 * Register Page Component
 * 
 * Handles new user registration with email and password.
 * Includes password confirmation and validation.
 */
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import './Auth.css';

function Register() {
  // Form state
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  
  // UI state
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  const navigate = useNavigate();

  /**
   * Validate form inputs
   */
  const validateForm = () => {
    setError('');
    
    if (!email.trim()) {
      setError('Email is required');
      return false;
    }
    
    // Email format check
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Please enter a valid email address');
      return false;
    }
    
    if (!password) {
      setError('Password is required');
      return false;
    }
    
    if (password.length < 8) {
      setError('Password must be at least 8 characters');
      return false;
    }
    
    // Password strength check
    const hasUppercase = /[A-Z]/.test(password);
    const hasLowercase = /[a-z]/.test(password);
    const hasNumber = /[0-9]/.test(password);
    
    if (!hasUppercase || !hasLowercase || !hasNumber) {
      setError('Password must contain uppercase, lowercase, and a number');
      return false;
    }
    
    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return false;
    }
    
    return true;
  };

  /**
   * Handle form submission
   */
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      // API call will be added later
      console.log('Register attempt:', { email });
      
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // On success, redirect to login
      navigate('/login');
      
    } catch (err) {
      setError('Registration failed. Please try again.');
      console.error('Register error:', err);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>Create Account</h1>
        <p className="auth-subtitle">Get started with Resume Agent</p>
        
        {error && (
          <div className="error-message">
            {error}
          </div>
        )}
        
        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              disabled={loading}
              autoComplete="email"
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Create a password"
              disabled={loading}
              autoComplete="new-password"
            />
            <small className="form-hint">
              8+ characters, uppercase, lowercase, and number
            </small>
          </div>
          
          <div className="form-group">
            <label htmlFor="confirmPassword">Confirm Password</label>
            <input
              id="confirmPassword"
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              placeholder="Confirm your password"
              disabled={loading}
              autoComplete="new-password"
            />
          </div>
          
          <button 
            type="submit" 
            className="auth-button"
            disabled={loading}
          >
            {loading ? 'Creating account...' : 'Create Account'}
          </button>
        </form>
        
        <p className="auth-footer">
          Already have an account?{' '}
          <Link to="/login">Sign in</Link>
        </p>
      </div>
    </div>
  );
}

export default Register;
```

#### Step 5: Create Auth CSS

Create file: `frontend/src/pages/Auth.css`

```css
/* Auth Pages Styles */

.auth-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.auth-card {
  background: white;
  border-radius: 16px;
  padding: 40px;
  width: 100%;
  max-width: 400px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.auth-card h1 {
  margin: 0 0 8px 0;
  font-size: 28px;
  color: #1a1a1a;
  text-align: center;
}

.auth-subtitle {
  margin: 0 0 24px 0;
  color: #666;
  text-align: center;
  font-size: 14px;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-size: 14px;
  font-weight: 500;
  color: #333;
}

.form-group input {
  padding: 12px 16px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.form-group input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-group input:disabled {
  background-color: #f5f5f5;
  cursor: not-allowed;
}

.form-hint {
  font-size: 12px;
  color: #888;
}

.error-message {
  background-color: #fee2e2;
  color: #dc2626;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 14px;
  margin-bottom: 16px;
}

.auth-button {
  padding: 14px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  margin-top: 8px;
}

.auth-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.auth-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.auth-footer {
  margin-top: 24px;
  text-align: center;
  color: #666;
  font-size: 14px;
}

.auth-footer a {
  color: #667eea;
  font-weight: 500;
}

.auth-footer a:hover {
  text-decoration: underline;
}
```

#### Step 6: Update App.jsx with New Pages

Update file: `frontend/src/App.jsx`

```jsx
/**
 * Main App Component
 */
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Import page components
import Login from './pages/Login';
import Register from './pages/Register';

// Placeholder components (will be proper pages later)
function Home() {
  return (
    <div style={{ 
      padding: '40px', 
      textAlign: 'center',
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      justifyContent: 'center',
      minHeight: '100vh'
    }}>
      <h1 style={{ fontSize: '48px', marginBottom: '16px' }}>Resume Agent</h1>
      <p style={{ fontSize: '20px', color: '#666', marginBottom: '32px' }}>
        AI-powered resume optimization
      </p>
      <div style={{ display: 'flex', gap: '16px' }}>
        <a 
          href="/login" 
          style={{
            padding: '12px 32px',
            background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
            color: 'white',
            borderRadius: '8px',
            textDecoration: 'none',
            fontWeight: 600
          }}
        >
          Login
        </a>
        <a 
          href="/register"
          style={{
            padding: '12px 32px',
            border: '2px solid #667eea',
            color: '#667eea',
            borderRadius: '8px',
            textDecoration: 'none',
            fontWeight: 600
          }}
        >
          Register
        </a>
      </div>
    </div>
  );
}

function Dashboard() {
  return (
    <div style={{ padding: '40px' }}>
      <h1>Dashboard</h1>
      <p>Upload your resume and job description here (coming soon)</p>
    </div>
  );
}

function Results() {
  return (
    <div style={{ padding: '40px' }}>
      <h1>Results</h1>
      <p>Your optimized resume will appear here (coming soon)</p>
    </div>
  );
}

function NotFound() {
  return (
    <div style={{ padding: '40px', textAlign: 'center' }}>
      <h1>404</h1>
      <p>Page not found</p>
      <a href="/">Go Home</a>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/results/:id" element={<Results />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

#### Step 7: Test the UI

```bash
cd frontend
npm run dev
```

Open browser and test:
1. http://localhost:5173 → Home page with styled buttons
2. http://localhost:5173/login → Login form
3. http://localhost:5173/register → Register form

**Test validation:**
- Submit empty form → Should show "Email is required"
- Enter invalid email → Should show "Please enter a valid email"
- Enter short password → Should show appropriate error
- Enter mismatched passwords (register) → Should show "Passwords do not match"

#### Step 8: Commit and PR

```bash
cd ..  # Back to resume-agent root

git add frontend/

git commit -m "AG-28: Create Login and Register UI components

- Add Login page with email/password form
- Add Register page with password confirmation
- Implement client-side validation
- Add error message display
- Add loading states
- Create Auth.css with modern styling
- Update App.jsx to use new components"

git push origin feature/AG-28-auth-ui
```

Open PR, merge, cleanup.

**Jira Update:** Move AG-28 to "Done"

---

## ✅ End of Day 3: Manual Verification

### Terminal Tests - Job Requirements Node

```bash
cd resume-agent/backend

# Set OpenAI API key
export OPENAI_API_KEY=your-key

# Run job requirements test
cd agent/nodes
python test_job_requirements.py

# Expected: All tests pass with extracted skills
```

### Terminal Tests - Scoring

```bash
cd resume-agent/backend/agent
python test_scoring.py

# Expected: All 4 tests pass
```

### Browser Tests - Auth UI

1. **Home Page (http://localhost:5173)**
   - Should see "Resume Agent" title
   - Login and Register buttons styled nicely

2. **Login Page (http://localhost:5173/login)**
   - Form with Email and Password fields
   - Try submitting empty → Error shows
   - Try invalid email → Error shows  
   - Try short password → Error shows
   - Fill valid credentials → Button shows "Signing in..." then redirects

3. **Register Page (http://localhost:5173/register)**
   - Form with Email, Password, Confirm Password
   - Test all validation messages
   - Link to Login page works

4. **Navigation**
   - Login page "Create one" link → Goes to Register
   - Register page "Sign in" link → Goes to Login

---

## 📁 Folder Structure After Day 3

```
resume-agent/
├── backend/
│   ├── agent/
│   │   ├── state.py          
│   │   ├── scoring.py        ✓ NEW
│   │   ├── test_scoring.py   ✓ NEW
│   │   └── nodes/
│   │       ├── job_requirements.py      ✓ NEW
│   │       └── test_job_requirements.py ✓ NEW
│   └── ...
├── frontend/
│   └── src/
│       ├── App.jsx           ✓ Updated
│       └── pages/
│           ├── Login.jsx     ✓ NEW
│           ├── Register.jsx  ✓ NEW
│           └── Auth.css      ✓ NEW
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-18: Resume Analysis Node | Dev 2 | 5 | ✓ Done |
| AG-19: ATS Scoring Node | Dev 3 | 5 | ✓ Done |
| AG-20: 3-Node Workflow Integration | Dev 1 | 3 | ✓ Done |

**Total Story Points Completed:** 13  
**Dev 1:** 3 points | **Dev 2:** 5 points | **Dev 3:** 5 points

---

**← [Day 2: Foundation](./day-02.md)** | **[Day 4: Remaining Nodes →](./day-04.md)**
