# Day 15: History UI

> **Date:** Sprint 2, Day 15  
> **Focus:** Build history page frontend  
> **Vertical Slice:** User can view past optimizations

---

## 🎯 Today's Goal

By end of today:
- ✅ History page component created (Dev 3)
- ✅ History list with navigation (Dev 1 - more frontend!)
- ✅ Detail view for past runs (Dev 2 - more frontend!)
- ✅ User can browse all past optimizations

---

## 📋 Jira Tasks

### Task AG-57: History Page Component

| Field | Value |
|-------|-------|
| **Task ID** | AG-57 |
| **Title** | History Page Component |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 5 |
| **Sprint** | Sprint 2 |
| **Priority** | High |

**Description:**
Create history page showing list of user's past optimization runs with basic info (date, scores, improvement).

**Acceptance Criteria:**
- [ ] File created: `frontend/src/pages/History.jsx`
- [ ] Calls GET /api/agent/history on load
- [ ] Displays run cards with: date, scores, delta
- [ ] Shows loading state
- [ ] Shows empty state if no runs
- [ ] PR merged

---

### Task AG-58: History List Navigation

| Field | Value |
|-------|-------|
| **Task ID** | AG-58 |
| **Title** | History List Navigation |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | Medium |

**Description:**
Add navigation from history page to view full details of any past run. Integrate with existing results page.

**Acceptance Criteria:**
- [ ] Each history item is clickable
- [ ] Click → navigate to /results/:id
- [ ] Results page loads run from API if not in localStorage
- [ ] Back navigation works correctly
- [ ] URL routing works
- [ ] PR merged

---

### Task AG-59: Detail View for Past Runs

| Field | Value |
|-------|-------|
| **Task ID** | AG-59 |
| **Title** | Detail View for Past Runs |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 3 |
| **Sprint** | Sprint 2 |
| **Priority** | Medium |

**Description:**
Update Results page to load past runs from API by ID. Show full details including cover letter if available.

**Acceptance Criteria:**
- [ ] Results page accepts :id param
- [ ] Calls GET /api/agent/run/:id if needed
- [ ] Displays all saved data from database
- [ ] Cover letter tab shows if cover_letter exists
- [ ] Handles loading and error states
- [ ] PR merged

---

## 👨‍💻 Dev 3: AG-26 - History Page

### Step 1: Create Branch

```bash
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-26-history-page
```

### Step 2: Update API Service

Add to `frontend/src/services/api.js`:

```javascript
/**
 * Get user's optimization history
 */
export async function getHistory() {
  const token = getToken();
  
  if (!token) {
    throw new Error('Not logged in');
  }

  const response = await fetch(`${API_URL}/api/agent/history`, {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });

  if (!response.ok) {
    throw new Error('Failed to fetch history');
  }

  return response.json();
}

/**
 * Get details of a specific run
 */
export async function getRunDetail(runId) {
  const token = getToken();
  
  const response = await fetch(`${API_URL}/api/agent/run/${runId}`, {
    headers: {
      ...(token && { 'Authorization': `Bearer ${token}` }),
    },
  });

  if (!response.ok) {
    throw new Error('Run not found');
  }

  return response.json();
}
```

### Step 3: Create History Page

Create file: `frontend/src/pages/History.jsx`:

```jsx
/**
 * Run History Page
 */
import { useState, useEffect } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { getHistory, isLoggedIn } from '../services/api';
import './History.css';

function History() {
  const [runs, setRuns] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');
  
  const navigate = useNavigate();

  useEffect(() => {
    if (!isLoggedIn()) {
      navigate('/login');
      return;
    }

    loadHistory();
  }, []);

  const loadHistory = async () => {
    try {
      const data = await getHistory();
      setRuns(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const formatDate = (dateStr) => {
    if (!dateStr) return 'N/A';
    const date = new Date(dateStr);
    return date.toLocaleDateString() + ' ' + date.toLocaleTimeString();
  };

  const getScoreClass = (delta) => {
    if (delta > 20) return 'excellent';
    if (delta > 10) return 'good';
    if (delta > 0) return 'fair';
    return 'neutral';
  };

  if (loading) {
    return (
      <div className="history-container">
        <div className="loading">Loading history...</div>
      </div>
    );
  }

  return (
    <div className="history-container">
      <header className="history-header">
        <h1>Optimization History</h1>
        <Link to="/dashboard" className="new-btn">New Optimization</Link>
      </header>

      {error && <div className="error-message">{error}</div>}

      {runs.length === 0 ? (
        <div className="empty-state">
          <h2>No optimizations yet</h2>
          <p>Run your first optimization to see history here.</p>
          <Link to="/dashboard" className="action-btn">Get Started</Link>
        </div>
      ) : (
        <div className="runs-list">
          {runs.map((run) => (
            <Link 
              key={run.id} 
              to={`/results/${run.id}`}
              className="run-card"
              onClick={() => {
                // Store for results page
                localStorage.setItem('currentRunId', run.id);
              }}
            >
              <div className="run-date">{formatDate(run.created_at)}</div>
              <div className="run-scores">
                <span className="score before">{run.ats_score_before?.toFixed(0) || '-'}</span>
                <span className="arrow">→</span>
                <span className="score after">{run.ats_score_after?.toFixed(0) || '-'}</span>
              </div>
              <div className={`run-delta ${getScoreClass(run.improvement_delta)}`}>
                +{run.improvement_delta?.toFixed(1) || 0} pts
              </div>
              <div className={`run-status ${run.status}`}>{run.status}</div>
            </Link>
          ))}
        </div>
      )}
    </div>
  );
}

export default History;
```

### Step 4: Create History CSS

Create file: `frontend/src/pages/History.css`:

```css
.history-container {
  min-height: 100vh;
  background: #f5f7fa;
  padding: 20px;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 800px;
  margin: 0 auto 30px;
}

.history-header h1 {
  font-size: 28px;
  margin: 0;
}

.new-btn {
  padding: 10px 20px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 600;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  background: white;
  border-radius: 12px;
  max-width: 400px;
  margin: 0 auto;
}

.runs-list {
  max-width: 800px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.run-card {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 20px;
  background: white;
  border-radius: 12px;
  text-decoration: none;
  color: inherit;
  transition: transform 0.2s, box-shadow 0.2s;
}

.run-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.run-date {
  color: #666;
  font-size: 14px;
  min-width: 160px;
}

.run-scores {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
}

.score.before { color: #f59e0b; }
.score.after { color: #10b981; }
.arrow { color: #999; }

.run-delta {
  padding: 4px 12px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 14px;
}

.run-delta.excellent { background: #d1fae5; color: #065f46; }
.run-delta.good { background: #dbeafe; color: #1e40af; }
.run-delta.fair { background: #fef3c7; color: #92400e; }
.run-delta.neutral { background: #f3f4f6; color: #4b5563; }

.run-status {
  margin-left: auto;
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  text-transform: uppercase;
}

.run-status.completed { background: #d1fae5; color: #065f46; }
.run-status.running { background: #dbeafe; color: #1e40af; }
.run-status.failed { background: #fee2e2; color: #dc2626; }
```

### Step 5: Update App.jsx

```jsx
import History from './pages/History';

// Add route:
<Route path="/history" element={<History />} />
```

### Step 6: Add Cover Letter Tab to Results

Update `Results.jsx` to add a cover letter tab:

```jsx
// Add tab button
<button 
  className={`tab ${activeTab === 'cover' ? 'active' : ''}`}
  onClick={() => setActiveTab('cover')}
>
  Cover Letter
</button>

// Add tab content
{activeTab === 'cover' && (
  <div className="cover-section">
    <div className="section-header">
      <h2>Generated Cover Letter</h2>
      <button onClick={() => handleCopy(result.cover_letter)} className="copy-btn">
        Copy
      </button>
    </div>
    {result.cover_letter ? (
      <pre className="cover-text">{result.cover_letter}</pre>
    ) : (
      <p>No cover letter generated for this run.</p>
    )}
  </div>
)}
```

### Step 7: Commit and PR

```bash
git add frontend/
git commit -m "AG-26: Create history page

- Show past optimization runs
- Link to view full results
- Add cover letter tab to results
- Add navigation between pages"

git push origin feature/AG-26-history-page
```

---

## ✅ Day 15: Verification

- [ ] History page shows past runs
- [ ] Click run → view results
- [ ] Cover letter tab works and can copy

---

## 📝 Daily Summary

| Task | Assignee | Points | Status |
|------|----------|--------|--------|
| AG-57: History Page Component | Dev 3 | 5 | ✓ Done |
| AG-58: History List Navigation | Dev 1 | 2 | ✓ Done |
| AG-59: Detail View for Past Runs | Dev 2 | 3 | ✓ Done |

**Total Story Points Completed:** 10  
**Dev 1:** 2 points | **Dev 2:** 3 points | **Dev 3:** 5 points

---

**← [Day 14](./day-14.md)** | **[Day 16: Loading UX →](./day-16.md)**
