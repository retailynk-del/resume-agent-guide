# Day 8: Agent API + Dashboard

> **Date:** Sprint 1, Day 8  
> **Focus:** Connect agent workflow to API, build dashboard UI  
> **Vertical Slice:** Slice 4 - Full flow working

---

## 🎯 Today's Goal

By end of today:
- ✅ Agent endpoint accepts resume + JD
- ✅ Dashboard UI with upload form
- ✅ User can submit and get results

---

## 📋 Morning: Add These Tasks to Jira

---

### Task AG-14: Agent Run Endpoint

| Field | Value |
|-------|-------|
| **Task ID** | AG-14 |
| **Title** | Agent Run Endpoint |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |

**Description:**
Create API endpoint that triggers the agent workflow and returns results.

**Acceptance Criteria:**
- [ ] POST /api/agent/run accepts JD + resume
- [ ] Returns optimization results
- [ ] Includes before/after scores
- [ ] Returns modified resume

---

### Task AG-18: Dashboard UI

| Field | Value |
|-------|-------|
| **Task ID** | AG-18 |
| **Title** | Dashboard Upload UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 1 |

**Description:**
Build dashboard with form to upload resume and job description. Shows loading state during processing.

**Acceptance Criteria:**
- [ ] Text areas for JD and resume
- [ ] Submit button triggers API
- [ ] Loading state during processing
- [ ] Redirect to results on success

---

## 👨‍💻 Dev 2 (Sinan): AG-14 - Agent Run Endpoint

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-14-agent-endpoint
```

**Jira:** Move AG-14 to "In Progress"

### Step 2: Create Agent Router

Create file: `backend/api/agent.py`

```python
"""
Agent API Endpoints

Handles resume optimization requests.
"""
import uuid
from typing import Optional

from fastapi import APIRouter, HTTPException, Depends, status
from pydantic import BaseModel

from api.auth import get_current_user_id
from agent.workflow import run_optimization

router = APIRouter(prefix="/api/agent", tags=["Agent"])


class OptimizationRequest(BaseModel):
    """Request model for resume optimization."""
    job_description: str
    resume: str
    
    class Config:
        json_schema_extra = {
            "example": {
                "job_description": "Looking for a Python developer...",
                "resume": "John Doe, Software Engineer..."
            }
        }


class OptimizationResponse(BaseModel):
    """Response model for optimization results."""
    run_id: str
    status: str
    ats_score_before: Optional[float]
    ats_score_after: Optional[float]
    improvement_delta: Optional[float]
    modified_resume: Optional[str]
    improvement_plan: Optional[dict]
    job_requirements: Optional[dict]


@router.post("/run", response_model=OptimizationResponse)
def run_agent(
    request: OptimizationRequest,
    user_id: str = Depends(get_current_user_id)
):
    """
    Run the resume optimization agent.
    
    Requires authentication (Bearer token).
    
    - **job_description**: The job posting text
    - **resume**: The user's resume text
    """
    # Validate input lengths
    if len(request.job_description.strip()) < 50:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Job description too short (minimum 50 characters)"
        )
    
    if len(request.resume.strip()) < 100:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Resume too short (minimum 100 characters)"
        )
    
    # Generate run ID
    run_id = str(uuid.uuid4())
    
    try:
        # Run the optimization workflow
        result = run_optimization(
            job_description=request.job_description,
            resume=request.resume,
            user_id=user_id,
            run_id=run_id
        )
        
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
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Optimization failed: {str(e)}"
        )


@router.post("/run-public", response_model=OptimizationResponse)
def run_agent_public(request: OptimizationRequest):
    """
    Run optimization without authentication (for demo purposes).
    
    In production, you might want to rate-limit this endpoint.
    """
    run_id = str(uuid.uuid4())
    
    if len(request.job_description.strip()) < 50:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Job description too short"
        )
    
    if len(request.resume.strip()) < 100:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Resume too short"
        )
    
    try:
        result = run_optimization(
            job_description=request.job_description,
            resume=request.resume,
            user_id="anonymous",
            run_id=run_id
        )
        
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
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Optimization failed: {str(e)}"
        )
```

### Step 3: Update Main App

Update `backend/main.py`:

```python
# Add import
from api.agent import router as agent_router

# Add after auth_router
app.include_router(agent_router)
```

### Step 4: Test the Endpoint

```bash
# Test with sample data
curl -X POST http://localhost:8000/api/agent/run-public \
  -H "Content-Type: application/json" \
  -d '{
    "job_description": "We are looking for a Senior Python Developer with 5+ years experience in Django and FastAPI. Must have PostgreSQL and Docker experience. AWS preferred.",
    "resume": "John Doe, Software Developer with 3 years of Python experience. Skills include Python, SQL, and web development. Previously worked at TechCorp building web applications."
  }'
```

### Step 5: Commit and PR

```bash
git add backend/api/ backend/main.py
git commit -m "AG-14: Create agent run endpoint

- POST /api/agent/run (authenticated)
- POST /api/agent/run-public (demo)
- Validate input lengths
- Return optimization results"

git push origin feature/AG-14-agent-endpoint
```

**Jira:** Move AG-14 to "Done"

---

## 👨‍💻 Dev 3 (Marva): AG-18 - Dashboard UI

### Step 1: Create Branch

```bash
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-18-dashboard
```

**Jira:** Move AG-18 to "In Progress"

### Step 2: Update API Service

Update `frontend/src/services/api.js` - add:

```javascript
/**
 * Run resume optimization
 */
export async function runOptimization(jobDescription, resume) {
  const token = getToken();
  
  const response = await fetch(`${API_URL}/api/agent/run-public`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      ...(token && { 'Authorization': `Bearer ${token}` }),
    },
    body: JSON.stringify({
      job_description: jobDescription,
      resume: resume,
    }),
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.detail || 'Optimization failed');
  }

  return data;
}
```

### Step 3: Create Dashboard Page

Create file: `frontend/src/pages/Dashboard.jsx`

```jsx
/**
 * Dashboard Page
 * 
 * Main interface for submitting resume optimization requests.
 */
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { runOptimization, isLoggedIn, logout } from '../services/api';
import './Dashboard.css';

function Dashboard() {
  const [jobDescription, setJobDescription] = useState('');
  const [resume, setResume] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Validate
    if (jobDescription.trim().length < 50) {
      setError('Job description must be at least 50 characters');
      return;
    }
    if (resume.trim().length < 100) {
      setError('Resume must be at least 100 characters');
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      const result = await runOptimization(jobDescription, resume);
      
      // Store result for results page
      localStorage.setItem('lastOptimization', JSON.stringify(result));
      
      // Navigate to results
      navigate(`/results/${result.run_id}`);
      
    } catch (err) {
      setError(err.message || 'Optimization failed. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  const handleLogout = () => {
    logout();
    navigate('/');
  };

  return (
    <div className="dashboard-container">
      <header className="dashboard-header">
        <h1>Resume Agent</h1>
        <div className="header-actions">
          {isLoggedIn() ? (
            <button onClick={handleLogout} className="logout-btn">
              Logout
            </button>
          ) : (
            <a href="/login">Login</a>
          )}
        </div>
      </header>

      <main className="dashboard-main">
        <div className="dashboard-intro">
          <h2>Optimize Your Resume</h2>
          <p>Paste your job description and resume below. Our AI will analyze and optimize your resume to better match the job requirements.</p>
        </div>

        {error && <div className="error-message">{error}</div>}

        <form onSubmit={handleSubmit} className="optimization-form">
          <div className="form-section">
            <label htmlFor="jobDescription">
              Job Description
              <span className="char-count">{jobDescription.length} characters</span>
            </label>
            <textarea
              id="jobDescription"
              value={jobDescription}
              onChange={(e) => setJobDescription(e.target.value)}
              placeholder="Paste the job description here...

Example:
We are looking for a Senior Software Engineer with 5+ years of experience in Python and Django. The ideal candidate will have experience with PostgreSQL, Docker, and AWS..."
              rows={10}
              disabled={loading}
            />
          </div>

          <div className="form-section">
            <label htmlFor="resume">
              Your Resume
              <span className="char-count">{resume.length} characters</span>
            </label>
            <textarea
              id="resume"
              value={resume}
              onChange={(e) => setResume(e.target.value)}
              placeholder="Paste your resume here...

Example:
John Smith
Software Engineer
john@email.com

EXPERIENCE
Software Developer at TechCorp (2020-Present)
- Developed web applications using Python
- Built REST APIs

SKILLS
Python, JavaScript, SQL, Git

EDUCATION
BS Computer Science, 2020"
              rows={12}
              disabled={loading}
            />
          </div>

          <button 
            type="submit" 
            className="submit-btn"
            disabled={loading}
          >
            {loading ? (
              <>
                <span className="spinner"></span>
                Optimizing (this may take 30-60 seconds)...
              </>
            ) : (
              'Optimize My Resume'
            )}
          </button>
        </form>
      </main>
    </div>
  );
}

export default Dashboard;
```

### Step 4: Create Dashboard CSS

Create file: `frontend/src/pages/Dashboard.css`

```css
/* Dashboard Styles */

.dashboard-container {
  min-height: 100vh;
  background: #f5f7fa;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 32px;
  background: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.dashboard-header h1 {
  font-size: 24px;
  margin: 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.logout-btn {
  padding: 8px 16px;
  background: transparent;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
}

.logout-btn:hover {
  background: #f5f5f5;
}

.dashboard-main {
  max-width: 900px;
  margin: 0 auto;
  padding: 40px 20px;
}

.dashboard-intro {
  text-align: center;
  margin-bottom: 32px;
}

.dashboard-intro h2 {
  font-size: 32px;
  margin-bottom: 12px;
  color: #1a1a1a;
}

.dashboard-intro p {
  color: #666;
  font-size: 16px;
  max-width: 600px;
  margin: 0 auto;
}

.error-message {
  background-color: #fee2e2;
  color: #dc2626;
  padding: 12px 16px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.optimization-form {
  background: white;
  padding: 32px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.form-section {
  margin-bottom: 24px;
}

.form-section label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
  margin-bottom: 8px;
  color: #333;
}

.char-count {
  font-weight: normal;
  font-size: 12px;
  color: #888;
}

.form-section textarea {
  width: 100%;
  padding: 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 14px;
  font-family: inherit;
  resize: vertical;
  transition: border-color 0.2s;
}

.form-section textarea:focus {
  outline: none;
  border-color: #667eea;
}

.form-section textarea:disabled {
  background-color: #f5f5f5;
}

.form-section textarea::placeholder {
  color: #aaa;
}

.submit-btn {
  width: 100%;
  padding: 16px 32px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  transition: transform 0.2s, box-shadow 0.2s;
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.submit-btn:disabled {
  opacity: 0.8;
  cursor: not-allowed;
}

/* Loading Spinner */
.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
```

### Step 5: Update App.jsx

```jsx
import Dashboard from './pages/Dashboard';

// Update route:
<Route path="/dashboard" element={<Dashboard />} />
```

### Step 6: Commit and PR

```bash
git add frontend/
git commit -m "AG-18: Create dashboard with optimization form

- Add runOptimization API function
- Create Dashboard page with JD and resume inputs
- Add loading state with spinner
- Store results and redirect to results page"

git push origin feature/AG-18-dashboard
```

**Jira:** Move AG-18 to "Done"

---

## ✅ Day 8: Manual Verification

### Terminal Tests

```bash
# Test agent endpoint
curl -X POST http://localhost:8000/api/agent/run-public \
  -H "Content-Type: application/json" \
  -d '{
    "job_description": "Looking for Python developer with Django experience. Must know PostgreSQL. AWS preferred. 5+ years needed.",
    "resume": "John Doe, Developer. 4 years Python. Skills: Python, Flask, MySQL. Experience at TechCorp building web apps and APIs."
  }'

# Should return optimization results with scores
```

### Browser Tests

1. **Open http://localhost:5173/dashboard**
2. Paste job description (50+ chars)
3. Paste resume (100+ chars)
4. Click "Optimize My Resume"
5. Wait for loading (30-60 seconds)
6. Should redirect to results page

### What Must Work

| Test | Expected |
|------|----------|
| Short input rejected | Error message |
| Valid input accepted | Shows loading |
| API returns results | Redirect to /results |

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-14: Agent Endpoint | Dev 2 | 5 | ✓ Done |
| AG-18: Dashboard UI | Dev 3 | 5 | ✓ Done |

**Total:** 10 points

---

**← [Day 7](./day-07.md)** | **[Day 9: Results Page →](./day-09.md)**
