# Day 17: Testing

> **Date:** Sprint 2, Day 17  
> **Focus:** Comprehensive testing of all features  
> **Vertical Slice:** Quality assurance and validation

---

## 🎯 Today's Goal

By end of today:
- ✅ Backend integration tests complete (Dev 2)
- ✅ Frontend E2E tests complete (Dev 3)
- ✅ Test documentation created (Dev 1)
- ✅ All critical flows validated

---

## 📋 Jira Tasks

### Task AG-63: Backend Integration Tests

| Field | Value |
|-------|-------|
| **Task ID** | AG-63 |
| **Title** | Backend Integration Tests |
| **Type** | Task |
| **Epic** | Testing & QA |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Write integration tests for all backend API endpoints. Test auth, agent runs, and history retrieval.

**Acceptance Criteria:**
- [ ] Test file created: `backend/tests/test_integration.py`
- [ ] Tests for auth endpoints (register, login)
- [ ] Tests for agent endpoint (run optimization)
- [ ] Tests for history endpoints
- [ ] All tests passing
- [ ] PR merged

---

### Task AG-64: Frontend E2E Tests

| Field | Value |
|-------|-------|
| **Task ID** | AG-64 |
| **Title** | Frontend E2E Tests |
| **Type** | Task |
| **Epic** | Testing & QA |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Create end-to-end tests for critical user flows using Playwright or similar. Test registration through results viewing.

**Acceptance Criteria:**
- [ ] E2E test framework set up
- [ ] Test: Register → Login → Dashboard flow
- [ ] Test: Submit optimization → View results
- [ ] Test: View history → View past run
- [ ] All tests passing
- [ ] PR merged

---

### Task AG-65: Test Documentation

| Field | Value |
|-------|-------|
| **Task ID** | AG-65 |
| **Title** | Test Documentation |
| **Type** | Task |
| **Epic** | Documentation |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | Medium |

**Description:**
Document testing strategy, test coverage, and how to run tests. Create testing guide for future developers.

**Acceptance Criteria:**
- [ ] File created: `docs/testing.md`
- [ ] Documents test strategy
- [ ] How to run backend tests
- [ ] How to run frontend tests
- [ ] Test coverage report
- [ ] PR merged

---

## 📝 Test Checklist

### Authentication Tests

| # | Test | Steps | Expected | ✓ |
|---|------|-------|----------|---|
| 1 | Register new user | POST /api/auth/register | 201, user_id returned | |
| 2 | Register duplicate | Same email again | 400, "Email already registered" | |
| 3 | Login valid | POST /api/auth/login | Token returned | |
| 4 | Login wrong password | Wrong password | 401, "Invalid credentials" | |
| 5 | Login non-existent | Random email | 401, "Invalid credentials" | |
| 6 | Access protected (no token) | GET /api/auth/me | 401 | |
| 7 | Access protected (valid) | With token | User info | |

### Agent Tests

| # | Test | Steps | Expected | ✓ |
|---|------|-------|----------|---|
| 8 | Run with short JD | <50 chars | 400, "too short" | |
| 9 | Run with short resume | <100 chars | 400, "too short" | |
| 10 | Run valid optimization | Valid inputs | Results with scores | |
| 11 | Score improved | Check before vs after | Delta > 0 | |
| 12 | Cover letter generated | Check response | cover_letter exists | |
| 13 | Run saved to history | GET /api/agent/history | Run appears | |

### Frontend Tests

| # | Test | Steps | Expected | ✓ |
|---|------|-------|----------|---|
| 14 | Home page loads | Navigate to / | Content visible | |
| 15 | Register flow | Fill form, submit | Success, redirect | |
| 16 | Login flow | Enter credentials | Dashboard redirect | |
| 17 | Dashboard validation | Empty submit | Error messages | |
| 18 | Optimization flow | Valid inputs | Progress, results | |
| 19 | Results display | After optimization | Scores visible | |
| 20 | Copy to clipboard | Click copy | "Copied!" message | |
| 21 | History page | Navigate | Past runs visible | |
| 22 | View past result | Click run | Full results | |

---

## 🐛 Bug Tracking Template

| ID | Description | Steps to Reproduce | Expected | Actual | Severity | Status |
|----|-------------|-------------------|----------|--------|----------|--------|
| BUG-1 | | | | | | |
| BUG-2 | | | | | | |

---

## 🔧 Common Bugs & Fixes

### Bug: CORS Error on Frontend

**Symptom:** Network error when calling API

**Fix:** Update `backend/main.py`:
```python
allowed_origins = os.getenv("ALLOWED_ORIGINS", "http://localhost:5173").split(",")
```

### Bug: Token Not Sent

**Symptom:** 401 on authenticated routes

**Fix:** Check `api.js` header:
```javascript
headers: {
  'Authorization': `Bearer ${token}`,  // Note: "Bearer " with space
}
```

### Bug: Empty History

**Symptom:** History returns empty even after runs

**Fix:** Ensure user_id matches in database query

---

## 📊 Test Results Summary

| Category | Passed | Failed | Skipped |
|----------|--------|--------|---------|
| Auth | /7 | | |
| Agent | /6 | | |
| Frontend | /9 | | |
| **Total** | /22 | | |

---

## 📝 Daily Summary

**Day 17:**
- Defined test checklist
- Ran auth tests
- Fixed 2 bugs

**Day 18:**
- Ran full integration tests
- Fixed remaining bugs
- All 22 tests passing

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-63: Backend Integration Tests | Dev 2 | 5 | ✓ Done |
| AG-64: Frontend E2E Tests | Dev 3 | 5 | ✓ Done |
| AG-65: Test Documentation | Dev 1 | 2 | ✓ Done |

**Total Story Points Completed:** 12  
**Dev 1:** 2 points | **Dev 2:** 5 points | **Dev 3:** 5 points

---

**← [Day 16](./day-16.md)** | **[Day 19: Polish →](./day-19.md)**
