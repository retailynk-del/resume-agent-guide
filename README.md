# Resume Optimization Agent - Developer Guide

> **Team:** Shabas (Dev 1), Sinan (Dev 2), Marva (Dev 3)  
> **Duration:** 21 Days (3 Sprints)  
> **Methodology:** Agile (Vertical Slices)

---

## 📅 Quick Links to Each Day

### Sprint 1 (Days 1-10): Core MVP

| Day | Focus | Link |
|-----|-------|------|
| 1 | Setup & LangGraph Tutorial | [Day 1](./days/day-01.md) |
| 2 | Agent State & Project Structure | [Day 2](./days/day-02.md) |
| 3 | First LangGraph Nodes (Pair) | [Day 3](./days/day-03.md) |
| 4 | Resume Analysis & Planning | [Day 4](./days/day-04.md) |
| 5 | Modification & Scoring Nodes | [Day 5](./days/day-05.md) |
| 6 | **Workflow Assembly (KEY!)** | [Day 6](./days/day-06.md) |
| 7 | Auth API & Frontend Connection | [Day 7](./days/day-07.md) |
| 8 | Agent API & Dashboard UI | [Day 8](./days/day-08.md) |
| 9 | Results Page & Testing | [Day 9](./days/day-09.md) |
| 10 | Sprint 1 Demo & Retro | [Day 10](./days/day-10.md) |

### Sprint 2 (Days 11-20): Enhanced Features

| Day | Focus | Link |
|-----|-------|------|
| 11 | Supabase Integration | [Day 11](./days/day-11.md) |
| 12 | Save Run History | [Day 12](./days/day-12.md) |
| 13 | Iterative Agent (Conditional Edges) | [Day 13](./days/day-13.md) |
| 14 | Cover Letter Generation | [Day 14](./days/day-14.md) |
| 15 | History UI Page | [Day 15](./days/day-15.md) |
| 16 | Loading UX Improvements | [Day 16](./days/day-16.md) |
| 17-18 | Integration Testing & Bug Fixes | [Day 17](./days/day-17.md) |
| 19 | Polish & Responsive Design | [Day 19](./days/day-19.md) |
| 20 | Sprint 2 Demo & Retro | [Day 20](./days/day-20.md) |

### Sprint 3 (Day 21+): Production Ready

| Day | Focus | Link |
|-----|-------|------|
| 21 | PDF Upload & Next Steps | [Day 21](./days/day-21.md) |

---

## Project Overview

### What We're Building
**Resume Optimization Agent** - An AI that automatically improves resumes to match job descriptions.

### User Flow
1. User uploads resume + job description
2. Agent analyzes and scores (before)
3. Agent plans and applies improvements
4. Agent re-scores (after) and iterates if needed
5. User downloads improved resume + cover letter

### Agile Vertical Slices

| Slice | Days | What Works End-to-End |
|-------|------|----------------------|
| 1 | 1-3 | Setup + LangGraph nodes running |
| 2 | 4-6 | Full agent workflow runs locally |
| 3 | 7-8 | Auth: Register → Login → Dashboard |
| 4 | 9-10 | Full MVP: Upload → Agent → Results |
| 5 | 11-15 | Persistence + History + Cover Letter |
| 6 | 16-20 | Polish + Testing + Sprint 2 Demo |
| 7 | 21+ | PDF Upload + Deployment |

---

## Technology Stack

| Component | Technology | Owner |
|-----------|------------|-------|
| Agent | LangGraph + GPT-4o-mini | All (Dev 1 leads) |
| Backend | FastAPI | Dev 2 (Sinan) |
| Frontend | React + Vite | Dev 3 (Marva) |
| Database | Supabase PostgreSQL | Dev 2 |
| CI/CD | GitHub Actions | Dev 3 |

---

## Jira & Git Workflow

### Jira Task Template
```
Title: AG-XX: [Task Name]
Epic: [Epic Name]
Assignee: [Developer]
Story Points: [1-8]
Sprint: Sprint 1/2/3

Description:
[What to do]

Acceptance Criteria:
- [ ] Criteria 1
- [ ] Criteria 2
```

### Git Workflow Per Task
```bash
# 1. START (Move Jira: To Do → In Progress)
git checkout dev-[name]
git pull origin main
git checkout -b feature/AG-XX-task-name

# 2. WORK (Commit every 30-60 min)
git add .
git commit -m "AG-XX: Description"

# 3. FINISH (Move Jira: In Progress → In Review)
git push origin feature/AG-XX-task-name
# Open PR on GitHub

# 4. AFTER MERGE (Move Jira: In Review → Done)
git checkout dev-[name]
git pull origin main
git branch -d feature/AG-XX-task-name
```

---

## Final Folder Structure

```
resume-agent/
├── .github/
│   └── workflows/
│       ├── backend-ci.yml
│       └── frontend-ci.yml
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   ├── .env
│   ├── api/
│   │   ├── auth.py
│   │   ├── agent.py
│   │   └── upload.py
│   ├── agent/
│   │   ├── state.py
│   │   ├── workflow.py
│   │   ├── scoring.py
│   │   └── nodes/
│   │       ├── job_requirements.py
│   │       ├── resume_analysis.py
│   │       ├── scoring.py
│   │       ├── planning.py
│   │       ├── modification.py
│   │       ├── decision.py
│   │       └── cover_letter.py
│   ├── database/
│   │   ├── connection.py
│   │   ├── init_db.py
│   │   └── models/
│   │       ├── user.py
│   │       └── run.py
│   └── utils/
│       └── pdf_parser.py
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   └── src/
│       ├── App.jsx
│       ├── main.jsx
│       ├── pages/
│       │   ├── Login.jsx
│       │   ├── Register.jsx
│       │   ├── Dashboard.jsx
│       │   ├── Results.jsx
│       │   └── History.jsx
│       ├── components/
│       │   ├── OptimizationProgress.jsx
│       │   ├── ErrorBoundary.jsx
│       │   └── Toast.jsx
│       └── services/
│           └── api.js
└── README.md
```

---

## Prerequisites (Before Day 1)

### Create Accounts

| Account | URL | Owner |
|---------|-----|-------|
| GitHub | https://github.com | All |
| Supabase | https://supabase.com | Dev 2 |
| OpenAI | https://platform.openai.com | Dev 1 |
| Vercel | https://vercel.com | Dev 3 |

### Install Tools

```bash
# Python
python --version  # Should be 3.10+

# Node
node --version    # Should be 18+

# Git
git --version
```

---

## Project Metrics (After 21 Days)

| Metric | Value |
|--------|-------|
| Total Story Points | 133 |
| Tasks Completed | 29 |
| API Endpoints | 8 |
| LangGraph Nodes | 8 |
| Database Models | 2 |
| React Pages | 5 |

---

**Start with [Day 1](./days/day-01.md) →**
