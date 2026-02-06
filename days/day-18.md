# Day 18: Bug Fixes

> **Date:** Sprint 2, Day 18  
> **Focus:** Fix issues found during testing  
> **Vertical Slice:** Stability and reliability

---

## 🎯 Today's Goal

By end of today:
- ✅ Backend bugs fixed (Dev 2)
- ✅ Frontend bugs fixed (Dev 3)
- ✅ Agent bugs fixed (Dev 1)
- ✅ All critical issues resolved

---

## 📋 Jira Tasks

### Task AG-60: Backend Bug Fixes

| Field | Value |
|-------|-------|
| **Task ID** | AG-60 |
| **Title** | Backend Bug Fixes |
| **Type** | Task |
| **Epic** | Bug Fixes |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Fix all backend issues found during testing. Address API errors, database issues, and auth problems.

**Acceptance Criteria:**
- [ ] Fix auth token expiration handling
- [ ] Fix database connection pooling
- [ ] Fix API error responses
- [ ] All backend tests passing
- [ ] PR merged

---

### Task AG-61: Frontend Bug Fixes

| Field | Value |
|-------|-------|
| **Task ID** | AG-61 |
| **Title** | Frontend Bug Fixes |
| **Type** | Task |
| **Epic** | Bug Fixes |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Fix all frontend issues found during testing. Address UI bugs, navigation issues, and state management problems.

**Acceptance Criteria:**
- [ ] Fix form validation edge cases
- [ ] Fix navigation state persistence
- [ ] Fix localStorage handling
- [ ] All frontend tests passing
- [ ] PR merged

---

### Task AG-62: Agent Bug Fixes

| Field | Value |
|-------|-------|
| **Task ID** | AG-62 |
| **Title** | Agent Bug Fixes |
| **Type** | Task |
| **Epic** | Bug Fixes |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Fix all LangGraph agent issues found during testing. Address workflow bugs, iteration problems, and scoring issues.

**Acceptance Criteria:**
- [ ] Fix iteration loop edge cases
- [ ] Fix score calculation accuracy
- [ ] Fix decision node logic
- [ ] All agent tests passing
- [ ] PR merged

---

## 🐛 Bug Fix Process

### 1. Triage Bugs from Day 17

Review all bugs found during testing and prioritize:

| Priority | Criteria |
|----------|----------|
| P1 Critical | Blocks core functionality |
| P2 High | Major feature impacted |
| P3 Medium | Minor issue, workaround exists |
| P4 Low | Cosmetic/nice-to-have |

### 2. Fix Assignment

Assign bugs to developers based on expertise:
- **Dev 1:** Agent/workflow bugs
- **Dev 2:** Backend/API bugs
- **Dev 3:** Frontend/UI bugs

### 3. Fix Workflow

```bash
# For each bug:
git checkout feature/bugfix-description
# Fix the issue
# Test the fix
git commit -m "AG-35b: Fix [description]"
git push origin feature/bugfix-description
# Open PR, get review, merge
```

---

## 📝 Documentation Checklist

### README.md Updates

- [ ] Quick start instructions
- [ ] Prerequisites list
- [ ] Environment variables documentation
- [ ] API endpoint summary
- [ ] Folder structure diagram

### API Documentation

```python
# FastAPI auto-generates docs at /docs
# Ensure all endpoints have:
- Summary
- Description
- Example request/response
- Error responses documented
```

### Code Comments

Review all files and ensure:
- [ ] All functions have docstrings
- [ ] Complex logic is commented
- [ ] No dead code remains
- [ ] All TODO comments resolved

---

## 🧹 Code Cleanup Checklist

### Backend

- [ ] Remove console.log / print statements
- [ ] Remove unused imports
- [ ] Format with black/ruff
- [ ] Type hints on all functions

```bash
cd backend
pip install black ruff
black .
ruff check . --fix
```

### Frontend

- [ ] Remove console.log statements
- [ ] Remove unused imports
- [ ] Format with prettier

```bash
cd frontend
npm run lint
npm run format  # if configured
```

---

## ✅ Day 18: Verification

### Pre-Flight Checklist

| Check | Status |
|-------|--------|
| All P1/P2 bugs fixed | ◻️ |
| All tests passing | ◻️ |
| README updated | ◻️ |
| API docs complete | ◻️ |
| Code formatted | ◻️ |
| No lint errors | ◻️ |

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-60: Backend Bug Fixes | Dev 2 | 3 | ✓ Done |
| AG-61: Frontend Bug Fixes | Dev 3 | 3 | ✓ Done |
| AG-62: Agent Bug Fixes | Dev 1 | 3 | ✓ Done |

**Total Story Points Completed:** 9  
**Dev 1:** 3 points | **Dev 2:** 3 points | **Dev 3:** 3 points

---

**← [Day 17](./day-17.md)** | **[Day 19: Polish →](./day-19.md)**
