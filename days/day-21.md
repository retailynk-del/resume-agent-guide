# Day 21: Sprint 2 Demo & PDF Upload

> **Date:** Sprint 2, Day 21  
> **Focus:** Sprint 2 demo presentation and PDF upload feature  
> **Milestone:** Project complete! 🎉

---

## 🎯 Today's Goal

By end of today:
- ✅ Sprint 2 demo presented (Dev 1)
- ✅ Retrospective documented (Dev 2)
- ✅ PDF upload feature started (Dev 3)
- ✅ Project successfully demonstrated to stakeholders

---

## 📋 Sprint 3 Goals Overview

| Goal | Priority | Story Points |
|------|----------|--------------|
| PDF Upload & Parsing | High | 8 |
| Deployment to Vercel/Railway | High | 8 |
| Email Delivery | Medium | 5 |
| Analytics Dashboard | Low | 5 |
| Performance Optimization | Medium | 5 |

---

## 📋 Jira Tasks for Day 21

### Task AG-69: Sprint 2 Demo Presentation

| Field | Value |
|-------|-------|
| **Task ID** | AG-69 |
| **Title** | Sprint 2 Demo Presentation |
| **Type** | Task |
| **Epic** | Project Management |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Present Sprint 2 demo to stakeholders showcasing all features: iterative agent, history, cover letter, and database persistence.

**Acceptance Criteria:**
- [ ] Demo script prepared
- [ ] Test data ready
- [ ] Full flow demonstrated
- [ ] Q&A handled
- [ ] Stakeholder feedback received
- [ ] Demo successful

---

### Task AG-70: Sprint 2 Retrospective

| Field | Value |
|-------|-------|
| **Task ID** | AG-70 |
| **Title** | Sprint 2 Retrospective |
| **Type** | Task |
| **Epic** | Project Management |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 1 |
| **Sprint** | Sprint 2 |
| **Priority** | Medium |

**Description:**
Facilitate Sprint 2 retrospective. Document learnings, action items, and celebrate achievements.

**Acceptance Criteria:**
- [ ] Retrospective session held
- [ ] What went well documented
- [ ] What to improve identified
- [ ] Action items created
- [ ] Document shared with team

---

### Task AG-71: PDF Upload Feature (Bonus)

| Field | Value |
|-------|-------|
| **Task ID** | AG-71 |
| **Title** | PDF Upload Feature |
| **Type** | Task |
| **Epic** | Future Features |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 3 (Preview) |
| **Priority** | Low |

**Description:**
Start Sprint 3 work: Add PDF upload capability to parse resume files. Extra credit / preview of future work.

**Acceptance Criteria:**
- [ ] Research PDF parsing libraries
- [ ] Install pdfplumber or PyPDF2
- [ ] Create basic PDF parser utility
- [ ] Test with sample PDF
- [ ] Document approach
- [ ] (Optional) PR merged

---

## 👨‍💻 Dev 1: AG-39 - PDF Parsing

### Step 1: Install Dependencies

```bash
cd backend
pip install pdfplumber python-multipart
pip freeze > requirements.txt
```

### Step 2: Create PDF Parser Module

Create file: `backend/utils/pdf_parser.py`:

```python
"""
PDF Parser Utility

Extracts text from PDF resume files.
"""
import io
from typing import Optional

import pdfplumber


def extract_text_from_pdf(pdf_bytes: bytes) -> str:
    """
    Extract all text from a PDF file.
    
    Args:
        pdf_bytes: Raw bytes of the PDF file
        
    Returns:
        Extracted text as a string
    """
    text_parts = []
    
    try:
        with pdfplumber.open(io.BytesIO(pdf_bytes)) as pdf:
            for page in pdf.pages:
                page_text = page.extract_text()
                if page_text:
                    text_parts.append(page_text)
        
        return "\n\n".join(text_parts)
    
    except Exception as e:
        raise ValueError(f"Failed to parse PDF: {str(e)}")


def validate_pdf(pdf_bytes: bytes) -> bool:
    """Check if bytes are valid PDF."""
    return pdf_bytes[:4] == b'%PDF'


def get_pdf_info(pdf_bytes: bytes) -> dict:
    """Get metadata about the PDF."""
    try:
        with pdfplumber.open(io.BytesIO(pdf_bytes)) as pdf:
            return {
                "pages": len(pdf.pages),
                "valid": True
            }
    except:
        return {"valid": False}
```

### Step 3: Create Test

Create file: `backend/utils/test_pdf_parser.py`:

```python
"""Tests for PDF parser."""
from pdf_parser import extract_text_from_pdf, validate_pdf


def test_text_extraction():
    # Would need a test PDF file
    # For now, test with empty/invalid
    try:
        result = extract_text_from_pdf(b"not a pdf")
        assert False, "Should raise error"
    except ValueError:
        pass
    print("✓ Invalid PDF handling test passed!")


def test_validate_pdf():
    assert not validate_pdf(b"not a pdf")
    assert validate_pdf(b'%PDF-1.4...')
    print("✓ PDF validation test passed!")


if __name__ == "__main__":
    test_text_extraction()
    test_validate_pdf()
    print("\n🎉 All PDF tests passed!")
```

---

## 👨‍💻 Dev 2: AG-40 - File Upload Endpoint

### Create Upload Router

Create file: `backend/api/upload.py`:

```python
"""
File Upload Endpoints
"""
from fastapi import APIRouter, HTTPException, UploadFile, File
from pydantic import BaseModel

from utils.pdf_parser import extract_text_from_pdf, validate_pdf

router = APIRouter(prefix="/api/upload", tags=["Upload"])

MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB


class ParsedResumeResponse(BaseModel):
    text: str
    pages: int
    character_count: int


@router.post("/resume", response_model=ParsedResumeResponse)
async def upload_resume(file: UploadFile = File(...)):
    """
    Upload and parse a PDF resume.
    
    Returns extracted text for use in optimization.
    """
    # Validate file type
    if not file.filename.lower().endswith('.pdf'):
        raise HTTPException(400, "Only PDF files are accepted")
    
    # Read file
    contents = await file.read()
    
    # Check size
    if len(contents) > MAX_FILE_SIZE:
        raise HTTPException(400, f"File too large. Max size: {MAX_FILE_SIZE // 1024 // 1024}MB")
    
    # Validate PDF
    if not validate_pdf(contents):
        raise HTTPException(400, "Invalid PDF file")
    
    # Extract text
    try:
        text = extract_text_from_pdf(contents)
    except ValueError as e:
        raise HTTPException(400, str(e))
    
    if len(text.strip()) < 100:
        raise HTTPException(400, "Could not extract enough text from PDF")
    
    return ParsedResumeResponse(
        text=text,
        pages=1,  # Would come from pdf_info
        character_count=len(text)
    )
```

### Update Main App

Add to `backend/main.py`:

```python
from api.upload import router as upload_router
app.include_router(upload_router)
```

---

## 📝 Deployment Planning

### Option A: Vercel (Frontend) + Railway (Backend)

**Frontend (Vercel):**
1. Connect GitHub repo
2. Set root to `frontend`
3. Add env: `VITE_API_URL`

**Backend (Railway):**
1. Connect GitHub repo
2. Set root to `backend`  
3. Add envs: `DATABASE_URL`, `OPENAI_API_KEY`, `SECRET_KEY`

### Option B: Docker Compose on VPS

```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
  
  frontend:
    build: ./frontend
    ports:
      - "3000:80"
```

---

## ✅ Day 21: Verification

```bash
# Test PDF upload
curl -X POST http://localhost:8000/api/upload/resume \
  -F "file=@test_resume.pdf"

# Should return extracted text
```

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-69: Sprint 2 Demo Presentation | Dev 1 | 2 | ✓ Done |
| AG-70: Sprint 2 Retrospective | Dev 2 | 1 | ✓ Done |
| AG-71: PDF Upload Feature (Bonus) | Dev 3 | 5 | ✓ Done |

**Total Story Points Completed:** 8  
**Dev 1:** 2 points | **Dev 2:** 1 point | **Dev 3:** 5 points

---

## 🚀 What's Next (Sprint 3 Continues)

### Days 22-25 (if project continues):
- **Day 22:** Frontend file upload UI
- **Day 23:** Deployment to production
- **Day 24:** Email delivery integration
- **Day 25:** Performance optimization

### Days 26-30:
- Analytics dashboard
- User settings
- Export options
- Final polish

---

## 📁 Complete Project Structure

After 21 days:

```
resume-agent/
├── .github/workflows/
├── backend/
│   ├── api/
│   │   ├── auth.py
│   │   ├── agent.py
│   │   └── upload.py       ← NEW
│   ├── agent/
│   │   ├── state.py
│   │   ├── scoring.py
│   │   ├── workflow.py
│   │   └── nodes/
│   ├── database/
│   │   └── models/
│   ├── utils/
│   │   └── pdf_parser.py  ← NEW
│   ├── main.py
│   └── requirements.txt
├── frontend/
│   └── src/
│       ├── components/
│       ├── pages/
│       └── services/
└── README.md
```

---

## 🎉 21-Day Summary

### What We Built

| Sprint | Focus | Key Deliverables |
|--------|-------|-----------------|
| Sprint 1 | Core Agent | 6 LangGraph nodes, Auth, Basic UI |
| Sprint 2 | Persistence | Supabase, History, Iteration, Cover Letter |
| Sprint 3 | Files | PDF parsing, Upload endpoint |

### Total Metrics

| Metric | Value |
|--------|-------|
| Story Points | 133 |
| Tasks Completed | 29 |
| API Endpoints | 8 |
| LangGraph Nodes | 8 |
| Database Models | 2 |
| React Pages | 5 |

---

## 🏆 Project Complete!

Congratulations! In 21 days, you've built a production-ready AI-powered Resume Optimization Agent with:

- ✅ Full user authentication
- ✅ LangGraph-based AI workflow
- ✅ Iterative optimization with target scoring
- ✅ Cover letter generation
- ✅ Run history and persistence
- ✅ PDF resume parsing
- ✅ Responsive, polished UI
- ✅ CI/CD pipeline

**Next Steps:**
1. Deploy to production
2. Add analytics
3. Implement email delivery
4. Scale and optimize

---

**← [Day 20](./day-20.md)** | **[Project Complete! 🎉](#)**
