# Day 16: Loading UX Improvements

> **Date:** Sprint 2, Day 6  
> **Focus:** Better loading states and progress feedback  
> **Vertical Slice:** Polish and UX

---

## 🎯 Today's Goal

By end of today:
- ✅ Progress steps during optimization
- ✅ Better error handling
- ✅ Smoother overall UX

---

## 📋 Jira Tasks

### Task AG-21: Loading Progress Steps

| Field | Value |
|-------|-------|
| **Task ID** | AG-21 |
| **Title** | Optimization Progress Steps |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |

**Description:**
Show users which step of optimization is happening.

---

### Task AG-33: Backend Progress SSE (Optional)

| Field | Value |
|-------|-------|
| **Task ID** | AG-33 |
| **Title** | Server-Side Progress Events |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 5 |

**Description:**
Add Server-Sent Events to stream progress updates.

---

## 👨‍💻 Dev 3: AG-21 - Progress Steps

### Step 1: Create Branch

```bash
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-21-progress-steps
```

### Step 2: Create Progress Component

Create file: `frontend/src/components/OptimizationProgress.jsx`:

```jsx
/**
 * Progress indicator for optimization steps
 */
import { useState, useEffect } from 'react';
import './OptimizationProgress.css';

const STEPS = [
  { id: 1, name: 'Analyzing Job', desc: 'Extracting requirements...' },
  { id: 2, name: 'Reviewing Resume', desc: 'Analyzing your experience...' },
  { id: 3, name: 'Calculating Score', desc: 'Measuring ATS compatibility...' },
  { id: 4, name: 'Planning Improvements', desc: 'Creating optimization plan...' },
  { id: 5, name: 'Optimizing Resume', desc: 'Applying improvements...' },
  { id: 6, name: 'Final Scoring', desc: 'Calculating new score...' },
  { id: 7, name: 'Writing Cover Letter', desc: 'Generating personalized letter...' },
];

function OptimizationProgress({ isRunning }) {
  const [currentStep, setCurrentStep] = useState(0);

  useEffect(() => {
    if (!isRunning) {
      setCurrentStep(0);
      return;
    }

    // Simulate progress (in production, use SSE for real progress)
    const interval = setInterval(() => {
      setCurrentStep((prev) => {
        if (prev >= STEPS.length) return prev;
        return prev + 1;
      });
    }, 5000); // ~5 seconds per step

    return () => clearInterval(interval);
  }, [isRunning]);

  if (!isRunning) return null;

  return (
    <div className="progress-overlay">
      <div className="progress-modal">
        <div className="progress-spinner"></div>
        <h2>Optimizing Your Resume</h2>
        <p className="progress-subtitle">This usually takes 30-60 seconds</p>
        
        <div className="progress-steps">
          {STEPS.map((step) => (
            <div 
              key={step.id}
              className={`step ${currentStep >= step.id ? 'active' : ''} ${currentStep === step.id ? 'current' : ''}`}
            >
              <div className="step-indicator">
                {currentStep > step.id ? '✓' : step.id}
              </div>
              <div className="step-content">
                <span className="step-name">{step.name}</span>
                {currentStep === step.id && (
                  <span className="step-desc">{step.desc}</span>
                )}
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

export default OptimizationProgress;
```

### Step 3: Create Progress CSS

Create file: `frontend/src/components/OptimizationProgress.css`:

```css
.progress-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.progress-modal {
  background: white;
  border-radius: 16px;
  padding: 40px;
  max-width: 400px;
  text-align: center;
}

.progress-spinner {
  width: 60px;
  height: 60px;
  border: 4px solid #e0e0e0;
  border-top-color: #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 24px;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.progress-modal h2 {
  margin: 0 0 8px;
  font-size: 24px;
}

.progress-subtitle {
  color: #666;
  margin: 0 0 32px;
}

.progress-steps {
  text-align: left;
}

.step {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 12px 0;
  opacity: 0.4;
  transition: opacity 0.3s;
}

.step.active {
  opacity: 1;
}

.step.current {
  background: #f0f4ff;
  margin: 0 -20px;
  padding: 12px 20px;
  border-radius: 8px;
}

.step-indicator {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: #e0e0e0;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 14px;
  flex-shrink: 0;
}

.step.active .step-indicator {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
}

.step-content {
  display: flex;
  flex-direction: column;
}

.step-name {
  font-weight: 600;
}

.step-desc {
  font-size: 13px;
  color: #666;
}
```

### Step 4: Update Dashboard.jsx

```jsx
import OptimizationProgress from '../components/OptimizationProgress';

// In the component, before form submit button:
<OptimizationProgress isRunning={loading} />
```

### Step 5: Commit

```bash
mkdir -p frontend/src/components
git add frontend/src/components/ frontend/src/pages/Dashboard.jsx
git commit -m "AG-21: Add optimization progress steps

- Visual step-by-step progress indicator
- Simulated timing (5s per step)
- Modal overlay during optimization"

git push origin feature/AG-21-progress-steps
```

---

## ✅ Day 16: Verification

- [ ] Progress modal appears during optimization
- [ ] Steps light up progressively
- [ ] Modal closes on completion/error

---

## 📝 Daily Summary

| Task | Points | Status |
|------|--------|--------|
| AG-21: Progress Steps | 3 | ✓ Done |

---

**← [Day 15](./day-15.md)** | **[Day 17: Integration Testing →](./day-17.md)**
