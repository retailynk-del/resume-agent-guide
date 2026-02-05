# Day 10: Sprint 1 Demo & Retrospective

> **Date:** Sprint 1, Day 10 (Final)  
> **Focus:** Demo to stakeholders, retrospective, Sprint 2 planning  
> **Milestone:** MVP Complete! 🎉

---

## 🎯 Today's Agenda

| Time | Activity |
|------|----------|
| 9:00 - 9:30 | Demo preparation |
| 9:30 - 10:00 | Sprint Demo |
| 10:00 - 10:30 | Stakeholder Q&A |
| 10:30 - 11:30 | Retrospective |
| 11:30 - 12:00 | Sprint 2 Planning Preview |
| PM | Documentation & cleanup |

---

## 📋 Morning: Jira Tasks

---

### Task AG-26: Sprint 1 Demo

| Field | Value |
|-------|-------|
| **Task ID** | AG-26 |
| **Title** | Sprint 1 Demo Presentation |
| **Assignees** | All Developers |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |

---

### Task AG-27: Retrospective

| Field | Value |
|-------|-------|
| **Task ID** | AG-27 |
| **Title** | Sprint 1 Retrospective |
| **Assignees** | All Developers |
| **Story Points** | 1 |
| **Sprint** | Sprint 1 |

---

## 🎬 Demo Preparation (9:00 - 9:30)

### Pre-Demo Checklist

**All developers:**

```bash
# 1. Pull latest code
cd resume-agent
git checkout main
git pull origin main

# 2. Start backend
cd backend
source venv/bin/activate  # or your virtualenv
pip install -r requirements.txt  # ensure all deps
uvicorn main:app --reload

# 3. In new terminal - Start frontend
cd frontend
npm install
npm run dev

# 4. Test the flow works
# Open http://localhost:5173
# Create a test account
# Run one optimization
# Verify results display
```

### Demo Script

**Presenter:** Dev 1 (Shabas)  
**Screen Share:** Dev 1's machine  
**Support:** Dev 2 & Dev 3 ready to answer questions

---

## 🎤 Demo Script (9:30 - 10:00)

### 1. Introduction (2 min)

> "Today we're demonstrating the Resume Optimization Agent - an AI-powered tool that helps job seekers improve their resumes to better match job descriptions.
>
> In Sprint 1, we built the core functionality: a LangGraph-based AI agent that analyzes job descriptions, scores resumes, creates an improvement plan, and rewrites the resume for better ATS matching."

### 2. Architecture Overview (3 min)

Show a simple diagram:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   React     │────▶│   FastAPI   │────▶│  LangGraph  │
│  Frontend   │     │   Backend   │     │    Agent    │
└─────────────┘     └─────────────┘     └─────────────┘
                                              │
                                              ▼
                                        ┌─────────────┐
                                        │   OpenAI    │
                                        │   GPT-4o    │
                                        └─────────────┘
```

> "The user interacts with a React frontend, which calls our FastAPI backend. The backend runs a LangGraph workflow with 6 nodes that process the resume through various AI analysis steps."

### 3. Live Demo (15 min)

**Step 1: Home Page**
> "This is our landing page. Users can register or login."

**Step 2: Registration**
> "Let's create a new account..."
- Fill in email and password
- Show success message

**Step 3: Login**
> "Now logging in with those credentials..."
- Show redirect to dashboard

**Step 4: Dashboard**
> "This is where the magic happens. Users paste their job description and resume."

- Paste prepared job description
- Paste prepared resume

> "Watch the character counters. We validate minimum lengths."

**Step 5: Submit & Wait**
> "When we click Optimize, the LangGraph agent runs. This takes 30-60 seconds as it makes several AI calls."

- Click submit
- Show loading state

> "Behind the scenes, six nodes are executing:
> 1. Extract job requirements
> 2. Analyze the resume
> 3. Calculate initial ATS score
> 4. Create improvement plan
> 5. Modify the resume
> 6. Calculate final score"

**Step 6: Results**
> "And here are our results!"

Show:
- Before score (e.g., 42)
- After score (e.g., 68)
- Improvement (+26 points!)

> "We improved the ATS score by 26 points!"

Show tabs:
- **Optimized Resume:** The rewritten resume
- **What Changed:** List of improvements made
- **Job Analysis:** Extracted requirements

**Step 7: Copy Feature**
> "Users can copy their optimized resume with one click."
- Click copy button
- Show "Copied!" message

### 4. Technical Highlights (5 min)

> "Some technical decisions worth noting:
>
> 1. **LangGraph State Machine:** We use TypedDict for type-safe state management. Each node receives state and returns updates to merge.
>
> 2. **ATS Scoring:** We built a custom scoring algorithm that evaluates keyword matches, skills presence, format quality, and section completeness.
>
> 3. **Decision Logging:** Every node logs its decisions, making the agent's reasoning transparent."

### 5. Sprint 1 Metrics (3 min)

| Metric | Value |
|--------|-------|
| Story Points Completed | 67 |
| Tasks Completed | 15 |
| Lines of Code (approx) | ~2,500 |
| API Endpoints | 4 |
| LangGraph Nodes | 6 |

---

## ❓ Q&A Session (10:00 - 10:30)

**Likely Questions:**

**Q: How accurate is the ATS scoring?**
> "Our current scoring is heuristic-based. In Sprint 2, we plan to add an LLM-based scoring option for more nuanced evaluation."

**Q: Can it handle PDF uploads?**
> "Not yet - that's planned for Sprint 3. Currently, users paste text directly."

**Q: What about data privacy?**
> "We use OpenAI's GPT-4o. Data is processed but not stored by OpenAI for training. In production, we'd add clear privacy policies."

**Q: How does iteration work?**
> "The current MVP is single-pass. Sprint 2 adds iterative refinement with conditional edges in LangGraph."

---

## 🔄 Retrospective (10:30 - 11:30)

### Format: Start / Stop / Continue

**Each team member writes sticky notes (5 min), then discuss (15 min each):**

#### ✅ What Worked Well (Continue)
- Daily standups
- Pair programming for complex tasks
- Clear Jira task definitions
- Testing before merging

#### 🛑 What Didn't Work (Stop)
- [Team to fill in]

#### 🚀 What To Improve (Start)
- [Team to fill in]

### Action Items

| Action | Owner | Due |
|--------|-------|-----|
| | | |
| | | |

---

## 📊 Sprint 1 Summary

### Completed User Stories

| Story | Points | Status |
|-------|--------|--------|
| As a user, I can register and login | 8 | ✅ Done |
| As a user, I can submit resume + JD | 10 | ✅ Done |
| As a user, I can see optimization results | 5 | ✅ Done |
| As a developer, the agent workflow runs E2E | 35 | ✅ Done |
| Infrastructure (CI/CD, DB) | 9 | ✅ Done |

**Total: 67 Story Points**

### What Ships

1. ✅ User authentication (register, login)
2. ✅ Resume optimization agent (6 nodes)
3. ✅ ATS scoring (before/after)
4. ✅ Results display with copy function
5. ✅ CI/CD pipeline (GitHub Actions)

### Known Limitations (for Sprint 2)

1. No persistent storage (in-memory users)
2. No run history
3. Single iteration only
4. No cover letter generation yet
5. Text-only input (no PDF)

---

## 📅 Sprint 2 Preview (11:30 - 12:00)

### Sprint 2 Goals

1. **Supabase Integration** - Store users and runs
2. **Iterative Refinement** - Agent loops until target score
3. **Cover Letter Generation** - Add new node
4. **Run History** - View past optimizations
5. **Improvements** - Based on retro feedback

### Sprint 2 Tasks Preview

| Day | Focus | Tasks |
|-----|-------|-------|
| 11 | Database | User model, Run history model |
| 12-13 | Agent Iteration | Conditional edges, loop logic |
| 14 | Cover Letter | New LangGraph node |
| 15-16 | UI Updates | History page, loading improvements |
| 17-18 | Testing | Integration, edge cases |
| 19 | Polish | Bug fixes, UX improvements |
| 20 | Demo Prep | Sprint 2 demo |

---

## 📝 Afternoon: Documentation

### Tasks

1. **Update README.md** with setup instructions
2. **Document API endpoints** in FastAPI docs
3. **Clean up code comments**
4. **Merge any outstanding PRs**

### README Template

```markdown
# Resume Optimization Agent

AI-powered resume optimization using LangGraph and GPT-4.

## Quick Start

### Prerequisites
- Python 3.11+
- Node.js 18+
- OpenAI API key

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # Add your OPENAI_API_KEY
uvicorn main:app --reload
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## API Documentation
Visit http://localhost:8000/docs

## Architecture
- Frontend: React + Vite
- Backend: FastAPI
- AI: LangGraph + GPT-4o-mini
```

---

## ✅ Day 10: Checklist

| Item | Status |
|------|--------|
| Demo preparation | ◻️ |
| Demo delivered | ◻️ |
| Q&A completed | ◻️ |
| Retrospective done | ◻️ |
| Action items recorded | ◻️ |
| Sprint 2 preview | ◻️ |
| Documentation updated | ◻️ |
| Outstanding PRs merged | ◻️ |

---

## 🎉 Sprint 1 Complete!

**Congratulations to the team!**

You've built a working AI agent that:
- Analyzes job descriptions
- Scores resumes against requirements
- Creates improvement plans
- Rewrites resumes for better ATS matching
- Shows measurable score improvement

**Next sprint starts Day 11!**

---

## 📁 Final Folder Structure After Sprint 1

```
resume-agent/
├── .github/
│   └── workflows/
│       ├── backend-ci.yml
│       └── frontend-ci.yml
├── backend/
│   ├── .env
│   ├── main.py
│   ├── requirements.txt
│   ├── api/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── agent.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── state.py
│   │   ├── scoring.py
│   │   ├── workflow.py
│   │   └── nodes/
│   │       ├── __init__.py
│   │       ├── job_requirements.py
│   │       ├── resume_analysis.py
│   │       ├── scoring.py
│   │       ├── planning.py
│   │       └── modification.py
│   └── database/
│       ├── __init__.py
│       └── connection.py
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── services/
│       │   └── api.js
│       └── pages/
│           ├── Auth.css
│           ├── Login.jsx
│           ├── Register.jsx
│           ├── Dashboard.css
│           ├── Dashboard.jsx
│           ├── Results.css
│           └── Results.jsx
└── README.md
```

---

## 📝 Sprint 1 Final Summary

| Metric | Value |
|--------|-------|
| Days | 10 |
| Story Points | 67 |
| Tasks Completed | 15 |
| PRs Merged | ~30 |
| Team Members | 3 |

**Sprint Velocity:** 67 points / 10 days = 6.7 points/day

---

**← [Day 9](./day-09.md)** | **[Day 11: Sprint 2 Begins →](./day-11.md)**
