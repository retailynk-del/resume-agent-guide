# Day 11: Sprint 2 Kickoff & Planning

> **Date:** Sprint 2, Day 11  
> **Focus:** Sprint 2 retrospective, planning, and goal alignment  
> **Vertical Slice:** Team retrospective complete, Sprint 2 goals defined, ready to implement iteration features

---

## 🎯 Today's Goal

By end of today:
- ✅ Sprint 1 retrospective completed with action items
- ✅ Sprint 2 goals defined and prioritized
- ✅ Team aligned on iteration feature (conditional edges)
- ✅ Cover letter generation planned
- ✅ Database models reviewed (already in Sprint 1!)
- ✅ Day 12 tasks assigned and ready

---

## 📋 Sprint 2 Planning Notes

**Sprint 2 Focus Areas:**
1. **Iteration Logic** - Add conditional edges to workflow for quality loops
2. **Cover Letter Generation** - Extend agent to generate tailored cover letters
3. **UI Enhancements** - History page, progress indicators
4. **Polish** - Error handling, loading states, user feedback

**Note:** Database persistence was completed in Sprint 1 (Days 6 & 8), so Sprint 2 can focus on advanced features!

---

## 📋 Jira Tasks for Day 11

### Task AG-46: Sprint 1 Retrospective

| Field | Value |
|-------|-------|
| **Task ID** | AG-46 |
| **Title** | Sprint 1 Retrospective |
| **Type** | Task |
| **Epic** | Project Management |
| **Assignee** | Dev 2 (Farhan) |
| **Story Points** | 1 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Facilitate Sprint 1 retrospective session. Document what went well, what needs improvement, and action items for Sprint 2.

**Acceptance Criteria:**
- [ ] Retrospective session facilitated
- [ ] Start/Stop/Continue notes documented
- [ ] Action items identified and prioritized
- [ ] Team feedback captured
- [ ] Retrospective document shared with team

---

### Task AG-47: Sprint 2 Planning

| Field | Value |
|-------|-------|
| **Task ID** | AG-47 |
| **Title** | Sprint 2 Planning Session |
| **Type** | Task |
| **Epic** | Project Management |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 1 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Lead Sprint 2 planning - define sprint goals, review task list, estimate story points, and ensure team alignment on priorities.

**Acceptance Criteria:**
- [ ] Sprint 2 goals clearly defined
- [ ] All tasks reviewed and understood
- [ ] Story points verified
- [ ] Sprint capacity assessed
- [ ] Team aligned on priorities
- [ ] Next tasks assigned

---

### Task AG-48: Database Architecture Review

| Field | Value |
|-------|-------|
| **Task ID** | AG-48 |
| **Title** | Database Architecture Review |
| **Type** | Task |
| **Epic** | Backend |
| **Assignee** | Team |
| **Story Points** | 1 |
| **Sprint** | Sprint 2 |
| **Priority** | Medium |

**Description:**
Review existing database models (User, AgentRun) created in Sprint 1. Verify schema supports Sprint 2 features (history page, iteration tracking).

**Acceptance Criteria:**
- [ ] User model reviewed (Day 6 AG-28)
- [ ] AgentRun model reviewed (Day 8 AG-37)
- [ ] Schema supports history queries
- [ ] Schema supports iteration tracking
- [ ] No changes needed for Sprint 2 features

---

## 🕘 9:00 AM - Sprint 1 Retrospective

### Dev 2 (Farhan) Facilitates

**Jira:** Move AG-46 to "In Progress"

### Retrospective Format: Start / Stop / Continue

#### ✅ What Went Well (Continue)

**Shahid:**
> "The daily standup format with task dependencies is working great. We rarely have blockers."

**Farhan:**
> "Code reviews are fast and helpful. The peer review system (Dev 1 → Dev 2 → Dev 3 → Dev 1) works well."

**Marva:**
> "The LangGraph framework was easier to work with than expected. The agent workflow is clean and modular."

**Team consensus:**
- Continue daily standups with dependency tracking
- Continue peer review rotation
- Continue vertical slice approach (1 feature end-to-end per day)

#### 🔴 What Didn't Go Well (Stop)

**Shahid:**
> "We spent too much time debugging CORS issues on Day 7. Should have enabled it from Day 3."

**Farhan:**
> "Initial Supabase setup took longer than expected. Documentation was unclear."

**Marva:**
> "Frontend styling took longer than backend. We underestimated CSS time."

**Team consensus:**
- Stop underestimating frontend tasks (add +1 to all UI story points)
- Stop deferring infrastructure setup (do it early)
- Stop skipping environment variable validation (add checks)

#### 💡 What to Try (Start)

**Shahid:**
> "Let's add automated tests to CI/CD pipeline. We're writing tests but not running them automatically."

**Farhan:**
> "We should add loading skeletons / placeholders for better UX during the 30-60 second optimization wait."

**Marva:**
> "Let's create a component library. We're repeating button/input styles across pages."

**Team consensus:**
- Start GitHub Actions for test automation
- Start adding loading skeletons for better UX
- Start shared component library (buttons, inputs, cards)

### Action Items for Sprint 2

| # | Action | Owner | Priority |
|---|--------|-------|----------|
| 1 | Add GitHub Actions CI workflow | Shahid | High |
| 2 | Create shared UI component library | Marva | Medium |
| 3 | Add environment variable validation | Farhan | Medium |
| 4 | Increase UI story point estimates by 20% | All | Low |
| 5 | Add loading skeletons to Dashboard/Results | Marva | Medium |

### Retrospective Document

Create `docs/sprint-1-retrospective.md`:

```markdown
# Sprint 1 Retrospective

**Date:** Day 11  
**Participants:** Shahid, Farhan, Marva

## Summary

Sprint 1 successfully delivered MVP with core resume optimization functionality. Team velocity was consistent at ~15 points/day. All Sprint 1 goals achieved.

## Continue
- Daily standups with dependency tracking
- Peer review rotation (1→2→3→1)
- Vertical slice approach

## Stop
- Underestimating frontend tasks
- Deferring infrastructure setup
- Skipping environment validation

## Start
- GitHub Actions CI/CD
- Loading skeletons for UX
- Shared component library

## Action Items
1. Add GitHub Actions CI workflow (Shahid) - Sprint 2
2. Create shared UI components (Marva) - Sprint 2
3. Add env var validation (Farhan) - Day 12
4. Adjust story point estimates (All) - Ongoing
5. Add loading skeletons (Marva) - Day 13-14

## Metrics
- **Sprint 1 Velocity:** 15 points/day average
- **Tasks Completed:** 37/37 (100%)
- **Demo Success:** ✅ All features working
- **Team Satisfaction:** 8.5/10
```

**Commit retrospective document:**

```bash
mkdir -p docs
cat > docs/sprint-1-retrospective.md << 'EOF'
[content above]
EOF

git add docs/sprint-1-retrospective.md
git commit -m "AG-46: Sprint 1 retrospective complete

- Start/Stop/Continue captured
- Action items identified for Sprint 2
- Team velocity measured
- Satisfaction metrics documented"

git push origin main
```

**Jira:** Move AG-46 to "Done"

---

## 🕙 10:00 AM - Sprint 2 Planning Session

### Dev 3 (Marva) Facilitates

**Jira:** Move AG-47 to "In Progress"

### Sprint 2 Goal Definition

**Primary Goal:**  
*Enable iterative resume optimization with quality loops and add cover letter generation*

**Secondary Goals:**
- Add conditional edges to workflow (score-based routing)
- Generate tailored cover letters from optimized resumes
- Display run history in UI
- Add progress indicators during optimization
- Improve error handling and user feedback

### Sprint 2 Task Breakdown

#### Week 1 (Days 11-15)
- **Day 11:** Sprint planning + retrospective ✓
- **Day 12:** Conditional edges in workflow (AG-49, AG-50)
- **Day 13:** Iteration testing + loop logic (AG-51, AG-52, AG-53)
- **Day 14:** Cover letter generation node (AG-54, AG-55, AG-56)
- **Day 15:** Cover letter UI + history page (AG-57, AG-58, AG-59, AG-59A)

#### Week 2 (Days 16-20)
- **Day 16:** Progress indicators + real-time updates (AG-60, AG-61, AG-62)
- **Day 17:** Error handling + retry logic (AG-63, AG-64, AG-65)
- **Day 18:** Polish + accessibility (AG-66, AG-67, AG-68)
- **Day 19:** Testing + bug fixes (AG-69, AG-70, AG-71)
- **Day 20:** Sprint 2 demo prep (AG-72, AG-73, AG-74)

#### Sprint 3 Preview (Days 21+)
- **Day 21:** Final polish, deployment prep, handoff documentation

### Story Point Review

| Epic | Tasks | Points | Days |
|------|-------|--------|------|
| Iteration Logic | AG-49 through AG-53 | 25 | 2 days |
| Cover Letter | AG-54 through AG-59A | 24 | 2 days |
| Progress UI | AG-60 through AG-62 | 15 | 1 day |
| Error Handling | AG-63 through AG-65 | 18 | 1 day |
| Polish | AG-66 through AG-68 | 15 | 1 day |
| Testing | AG-69 through AG-71 | 18 | 1 day |
| Demo | AG-72 through AG-74 | 5 | 1 day |
| **Total** | **35 tasks** | **120 points** | **9 days** |

### Capacity Assessment

- **Team size:** 3 developers
- **Sprint length:** 10 days (Days 11-20)
- **Expected velocity:** 15 points/day × 10 days = 150 points
- **Planned work:** 120 points
- **Buffer:** 30 points (20%)

**Conclusion:** Sprint 2 is properly scoped with healthy buffer for unknowns.

### Risk Analysis

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Conditional edges complex | Medium | High | Start Day 12, pair programming |
| Cover letter quality varies | High | Medium | Test with multiple samples, iteration |
| Real-time progress difficult | Medium | Medium | Use polling fallback if WebSocket hard |
| OpenAI rate limits | Low | High | Add retry logic, queue system |

### Sprint 2 Planning Document

Create `docs/sprint-2-plan.md`:

```markdown
# Sprint 2 Plan

**Duration:** Days 11-20 (10 days)  
**Goal:** Iterative optimization with cover letter generation

## Sprint Goals

### Primary
✅ Add conditional edges for quality-based iteration  
✅ Generate tailored cover letters  
✅ Display optimization history

### Secondary
✅ Progress indicators during optimization  
✅ Error handling and retry logic  
✅ UI polish and accessibility

## Task Assignment - Day 12

| Task | Assignee | Points | Description |
|------|----------|--------|-------------|
| AG-49 | Shahid | 5 | Add should_iterate function |
| AG-50 | Farhan | 5 | Conditional edge routing logic |
| AG-51 | Marva | 3 | Test iteration with poor resumes |

## Success Metrics

- Iteration improves scores by additional 5-10 points
- Cover letters generated in < 15 seconds
- History page loads in < 1 second
- Zero crashes during optimization
- All tests passing

## Definition of Done

- All 35 tasks completed
- Test coverage > 80%
- Demo script tested
- Documentation updated
- Sprint 2 demo successful
```

**Commit planning document:**

```bash
cat > docs/sprint-2-plan.md << 'EOF'
[content above]
EOF

git add docs/sprint-2-plan.md
git commit -m "AG-47: Sprint 2 planning complete

- Sprint goals defined
- 35 tasks organized across 10 days
- Story points: 120 (within capacity)
- Risk analysis documented
- Success metrics established"

git push origin main
```

**Jira:** Move AG-47 to "Done"

---

## 🕚 11:00 AM - Database Architecture Review

### All Team: AG-48

**Jira:** Move AG-48 to "In Progress"

### Existing Database Models (Created Sprint 1)

#### User Model (Day 6 - AG-28)

**Location:** `backend/models/user.py`

**Schema:**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

**Purpose:** Authentication and user identification  
**Sprint 2 Usage:** Link optimization runs and history to users

---

#### AgentRun Model (Day 8 - AG-37)

**Location:** `backend/models/agent_run.py`

**Schema:**
```sql
CREATE TABLE agent_runs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id),
    job_description TEXT NOT NULL,
    original_resume TEXT NOT NULL,
    modified_resume TEXT,
    ats_score_before FLOAT,
    ats_score_after FLOAT,
    improvement_delta FLOAT,
    job_requirements JSONB,
    resume_analysis JSONB,
    improvement_plan JSONB,
    iteration_count INTEGER DEFAULT 0,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP
);

CREATE INDEX idx_agent_runs_user_id ON agent_runs(user_id);
CREATE INDEX idx_agent_runs_created_at ON agent_runs(created_at DESC);
```

**Purpose:** Store all optimization results  
**Sprint 2 Usage:** 
- Display history page (query by user_id, order by created_at)
- Track iteration count for conditional logic
- Store cover letter in new column (add later)

---

### Schema Verification in Supabase

**Shahid:** *Opens Supabase dashboard*

"Let me check the tables..."

1. **Go to:** https://supabase.com → Your Project → Table Editor
2. **Verify `users` table:**
   - Columns: id (uuid), email (varchar), password_hash (text), created_at (timestamp)
   - Index on email ✅
3. **Verify `agent_runs` table:**
   - All columns present ✅
   - Foreign key to users ✅
   - Indexes on user_id and created_at ✅

**Farhan:** "Do we need any schema changes for Sprint 2?"

**Marva:** "We'll need to track iterations and store cover letters. Let me check the model..."

*Checks [backend/models/agent_run.py](../backend/models/agent_run.py)*

**Marva:** "We already have `iteration_count` field! And we can add `cover_letter TEXT` column on Day 14."

**Team consensus:** ✅ Current schema supports Sprint 2 features. Only need to add one column later.

---

### Database Migration Plan

#### Day 14: Add Cover Letter Column

```sql
-- Run this migration on Day 14
ALTER TABLE agent_runs 
ADD COLUMN cover_letter TEXT;
```

**Update AgentRun model:**
```python
class AgentRun(Base):
    # ... existing fields ...
    cover_letter = Column(Text, nullable=True)  # Add this
```

**No migration needed for Days 11-13!**

---

### Query Planning for Sprint 2 Features

#### History Page (Day 15)

```python
# Get user's optimization history
runs = db.query(AgentRun)\
    .filter(AgentRun.user_id == current_user_id)\
    .order_by(AgentRun.created_at.desc())\
    .limit(50)\
    .all()
```

**Performance:** ✅ Index on `(user_id, created_at)` will make this fast

---

#### Iteration Tracking (Day 12)

```python
# Update iteration count after each loop
run.iteration_count += 1
db.commit()
```

**Performance:** ✅ Simple integer increment

---

### Database Review Complete

**Create review document:**

```bash
cat > docs/database-review-sprint2.md << 'EOF'
# Database Review - Sprint 2

**Date:** Day 11  
**Focus:** Verify existing schema supports Sprint 2 features

## Current Schema

### Users Table
- Created: Day 6 (AG-28)
- Status: ✅ No changes needed

### Agent_Runs Table
- Created: Day 8 (AG-37)
- Status: ✅ Ready for Sprint 2
- Migration needed: Add cover_letter column on Day 14

## Sprint 2 Schema Changes

### Day 14: Add Cover Letter Column
```sql
ALTER TABLE agent_runs ADD COLUMN cover_letter TEXT;
```

## Performance Verification
- ✅ History query uses index
- ✅ Iteration tracking is simple update
- ✅ No bottlenecks identified

## Conclusion
Database is ready for Sprint 2 features. Only one minor migration needed on Day 14.
EOF

git add docs/database-review-sprint2.md
git commit -m "AG-48: Database architecture review complete

- User and AgentRun models verified
- Schema supports Sprint 2 features
- Migration plan documented (Day 14)
- Query performance validated"

git push origin main
```

**Jira:** Move AG-48 to "Done"

---

## 🕐 1:00 PM - Team Lunch & Informal Discussion

### Casual Conversation

**Shahid:** "I'm excited about the iteration logic! It's like giving the agent a 'try again' button."

**Farhan:** "Yeah, and conditional edges are a core LangGraph feature. Should be straightforward to implement."

**Marva:** "The cover letter feature will be really valuable for users. I've spent hours writing custom cover letters before!"

**Team:** *Discusses conditional edge routing, shares LangGraph documentation links, plans Day 12 kickoff*

---

## 🕑 2:00 PM - Day 12 Task Assignment

### Task Handoff

**Shahid (Dev 1):**
- **Assigned:** AG-49 - Add should_iterate decision function
- **Points:** 5
- **Branch:** `feature/AG-49-should-iterate`

**Farhan (Dev 2):**
- **Assigned:** AG-50 - Add conditional edges to workflow  
- **Points:** 5
- **Branch:** `feature/AG-50-conditional-edges`
- **Depends on:** AG-49

**Marva (Dev 3):**
- **Assigned:** AG-51 - Test iteration with edge cases
- **Points:** 3
- **Branch:** `feature/AG-51-iteration-tests`
- **Depends on:** AG-50

---

## ✅ Day 11: Verification Checklist

| # | Item | Status |
|---|------|--------|
| 1 | Sprint 1 retrospective document created | ✅ |
| 2 | Start/Stop/Continue captured | ✅ |
| 3 | Action items documented | ✅ |
| 4 | Sprint 2 goals defined | ✅ |
| 5 | Task breakdown reviewed | ✅ |
| 6 | Story points verified | ✅ |
| 7 | Sprint capacity assessed | ✅ |
| 8 | Database schema reviewed | ✅ |
| 9 | Migration plan documented | ✅ |
| 10 | Day 12 tasks assigned | ✅ |

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-46: Sprint 1 Retrospective | Dev 2 (Farhan) | 1 | ✅ Done |
| AG-47: Sprint 2 Planning | Dev 3 (Marva) | 1 | ✅ Done |
| AG-48: Database Review | Team | 1 | ✅ Done |

**Total Story Points Completed:** 3  
**Dev 1 (Shahid):** 0 points (participated in reviews) | **Dev 2 (Farhan):** 1 point | **Dev 3 (Marva):** 1 point | **Team:** 1 point

---

## 🎉 Sprint 2 Kickoff Complete!

Day 11 achievements:
- ✅ Sprint 1 retrospective with actionable insights
- ✅ Sprint 2 goals clearly defined
- ✅ 35 tasks organized across 10 days
- ✅ Database architecture verified ready for Sprint 2
- ✅ Team aligned and ready for iteration features

**Tomorrow: Conditional Edges & Iteration Logic! 🔄**

---

**← [Day 10](./day-10.md)** | **[Day 12: Conditional Edges →](./day-12.md)**
        nullable=False
    )
    
    def __repr__(self):
        return f"<User {self.email}>"
    
    def to_dict(self):
        """Convert to dictionary for JSON serialization."""
        return {
            "id": str(self.id),
            "email": self.email,
            "created_at": self.created_at.isoformat()
        }
```

### Step 4: Update Models Init

Update file: `backend/database/models/__init__.py`

```python
"""Database models."""
from .user import User

__all__ = ["User"]
```

### Step 5: Create Database Migration Script

Create file: `backend/database/init_db.py`

```python
"""
Database Initialization Script

Run this once to create tables in Supabase.
"""
from connection import engine, Base
from models.user import User

def init_database():
    """Create all tables."""
    if engine is None:
        print("❌ DATABASE_URL not configured")
        return False
    
    try:
        Base.metadata.create_all(bind=engine)
        print("✓ Database tables created successfully")
        return True
    except Exception as e:
        print(f"❌ Error creating tables: {e}")
        return False


if __name__ == "__main__":
    from dotenv import load_dotenv
    load_dotenv()
    init_database()
```

### Step 6: Update Auth Router

Update file: `backend/api/auth.py`

```python
"""
Authentication API Endpoints

Uses Supabase PostgreSQL for user storage.
"""
import os
from datetime import datetime, timedelta
from typing import Optional
import uuid

from fastapi import APIRouter, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
from jose import jwt, JWTError
from sqlalchemy.orm import Session

from database.connection import get_db, SessionLocal
from database.models.user import User

router = APIRouter(prefix="/api/auth", tags=["Authentication"])

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key-change-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_HOURS = 24


# Pydantic models (same as before)
class UserRegister(BaseModel):
    email: EmailStr
    password: str

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    user_id: str

class UserResponse(BaseModel):
    id: str
    email: str

class MessageResponse(BaseModel):
    message: str
    user_id: Optional[str] = None


# Helper functions
def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(hours=ACCESS_TOKEN_EXPIRE_HOURS)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None


# API Endpoints
@router.post("/register", response_model=MessageResponse, status_code=201)
def register(user: UserRegister, db: Session = Depends(get_db)):
    """Register a new user."""
    # Check if email exists
    existing = db.query(User).filter(User.email == user.email.lower()).first()
    if existing:
        raise HTTPException(
            status_code=400,
            detail="Email already registered"
        )
    
    if len(user.password) < 6:
        raise HTTPException(
            status_code=400,
            detail="Password must be at least 6 characters"
        )
    
    # Create user
    new_user = User(
        email=user.email.lower(),
        password_hash=hash_password(user.password)
    )
    
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    
    return MessageResponse(
        message="User registered successfully",
        user_id=str(new_user.id)
    )


@router.post("/login", response_model=TokenResponse)
def login(user: UserLogin, db: Session = Depends(get_db)):
    """Login and receive JWT token."""
    db_user = db.query(User).filter(User.email == user.email.lower()).first()
    
    if not db_user:
        raise HTTPException(
            status_code=401,
            detail="Invalid email or password"
        )
    
    if not verify_password(user.password, db_user.password_hash):
        raise HTTPException(
            status_code=401,
            detail="Invalid email or password"
        )
    
    access_token = create_access_token(
        data={"sub": str(db_user.id), "email": db_user.email}
    )
    
    return TokenResponse(
        access_token=access_token,
        token_type="bearer",
        expires_in=ACCESS_TOKEN_EXPIRE_HOURS * 3600,
        user_id=str(db_user.id)
    )


@router.get("/me", response_model=UserResponse)
def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
):
    """Get current user info."""
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    user_id = payload.get("sub")
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    
    return UserResponse(id=str(user.id), email=user.email)


def get_current_user_id(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> str:
    """Dependency to get current user ID from token."""
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return payload.get("sub")
```

### Step 7: Initialize Database

```bash
cd backend/database
python init_db.py

# Expected: ✓ Database tables created successfully
```

### Step 8: Test

```bash
cd backend
uvicorn main:app --reload

# Test registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "db-test@test.com", "password": "password123"}'

# Test login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "db-test@test.com", "password": "password123"}'
```

### Step 9: Commit and PR

```bash
git add backend/database/ backend/api/auth.py
git commit -m "AG-40: Migrate auth to Supabase

- Create User SQLAlchemy model
- Update auth endpoints to use database
- Add database init script
- All existing auth tests pass"

git push origin feature/AG-40-user-model
```

**Jira:** Move AG-40 to "Done"

---

## 👨‍💻 Dev 1 (Shabas): AG-41 - Run History Model

### Step 1: Create Branch

```bash
git checkout dev-shabas && git pull origin main
git checkout -b feature/AG-41-run-model
```

**Jira:** Move AG-41 to "In Progress"

### Step 2: Create Run Model

Create file: `backend/database/models/run.py`

```python
"""
Agent Run Model

Stores optimization run history.
"""
from datetime import datetime
import uuid

from sqlalchemy import Column, String, DateTime, Float, Text, ForeignKey, JSON
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.orm import relationship

from database.connection import Base


class AgentRun(Base):
    """Stores each optimization run."""
    
    __tablename__ = "agent_runs"
    
    id = Column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4
    )
    
    user_id = Column(
        UUID(as_uuid=True),
        ForeignKey("users.id"),
        nullable=True,  # Allow anonymous runs
        index=True
    )
    
    job_description = Column(Text, nullable=False)
    original_resume = Column(Text, nullable=False)
    modified_resume = Column(Text)
    
    ats_score_before = Column(Float)
    ats_score_after = Column(Float)
    improvement_delta = Column(Float)
    
    job_requirements = Column(JSON)
    improvement_plan = Column(JSON)
    
    status = Column(
        String(20),
        default="pending"  # pending, running, completed, failed
    )
    
    created_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
    
    def to_dict(self):
        return {
            "id": str(self.id),
            "user_id": str(self.user_id) if self.user_id else None,
            "ats_score_before": self.ats_score_before,
            "ats_score_after": self.ats_score_after,
            "improvement_delta": self.improvement_delta,
            "status": self.status,
            "created_at": self.created_at.isoformat() if self.created_at else None
        }
    
    def to_full_dict(self):
        """Include full content for detail view."""
        data = self.to_dict()
        data.update({
            "job_description": self.job_description,
            "original_resume": self.original_resume,
            "modified_resume": self.modified_resume,
            "job_requirements": self.job_requirements,
            "improvement_plan": self.improvement_plan
        })
        return data
```

### Step 3: Update Models Init

Update file: `backend/database/models/__init__.py`

```python
"""Database models."""
from .user import User
from .run import AgentRun

__all__ = ["User", "AgentRun"]
```

### Step 4: Update Init Script

Update file: `backend/database/init_db.py`

```python
"""Database initialization."""
from connection import engine, Base
from models.user import User
from models.run import AgentRun

def init_database():
    """Create all tables."""
    if engine is None:
        print("❌ DATABASE_URL not configured")
        return False
    
    try:
        Base.metadata.create_all(bind=engine)
        print("✓ All database tables created")
        return True
    except Exception as e:
        print(f"❌ Error: {e}")
        return False

if __name__ == "__main__":
    from dotenv import load_dotenv
    load_dotenv()
    init_database()
```

### Step 5: Run Migration

```bash
cd backend/database
python init_db.py
# Should show: ✓ All database tables created
```

### Step 6: Commit and PR

```bash
git add backend/database/
git commit -m "AG-41: Create AgentRun model

- Store optimization runs in database
- Include all scores and content
- Add status tracking
- Link to user (optional for anonymous)"

git push origin feature/AG-41-run-model
```

**Jira:** Move AG-41 to "Done"

---

## ✅ Day 11: Manual Verification

### Database Verification (Supabase Dashboard)

1. Login to Supabase: https://supabase.com
2. Open your project
3. Go to Table Editor
4. Verify `users` table exists with correct columns
5. Verify `agent_runs` table exists

### Terminal Tests

```bash
# Test user registration still works
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "sprint2-test@test.com", "password": "Test1234"}'

# Check Supabase - user should appear in users table
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-45: Demo Preparation | Dev 1 | 2 | ✓ Done |
| AG-46: Retrospective Notes | Dev 2 | 1 | ✓ Done |
| AG-47: Sprint 2 Planning | Dev 3 | 1 | ✓ Done |

**Total Story Points Completed:** 4  
**Dev 1:** 2 points | **Dev 2:** 1 point | **Dev 3:** 1 point

---

**← [Day 10](./day-10.md)** | **[Day 12: Save Run History →](./day-12.md)**
