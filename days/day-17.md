# Day 17-18: Integration Testing & Bug Fixes

> **Date:** Sprint 2, Days 7-8  
> **Focus:** Comprehensive testing and bug fixes  
> **Vertical Slice:** Quality assurance

---

## 🎯 Goal

By end of Day 18:
- ✅ All user flows tested
- ✅ Edge cases handled
- ✅ Known bugs fixed

---

## 📋 Jira Tasks

### Task AG-35: Integration Test Suite

| Field | Value |
|-------|-------|
| **Task ID** | AG-35 |
| **Title** | Integration Test Suite |
| **Assignees** | All Developers |
| **Story Points** | 8 |

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

| Task | Points | Status |
|------|--------|--------|
| AG-35: Integration Tests | 8 | ✓ Done |

---

**← [Day 16](./day-16.md)** | **[Day 19: Polish →](./day-19.md)**
