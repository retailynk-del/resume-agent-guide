# Day 7: Auth Frontend

> **Date:** Sprint 1, Day 7  
> **Focus:** User authentication UI (login + register pages)  
> **Vertical Slice:** Full auth flow works in UI

---

## 🎯 Today's Goal

By end of today:
- ✅ Login page component ready (Dev 3)
- ✅ Register page component ready (Dev 3)
- ✅ Auth API service module created (Dev 1 - learning frontend!)
- ✅ Protected routes working (Dev 2 - cross-training!)
- ✅ Full flow: Register in UI → Login → Get token → Access dashboard

---

## 📋 Morning: Add These Tasks to Jira

---

### Task AG-32: Login Page Component

| Field | Value |
|-------|-------|
| **Task ID** | AG-32 |
| **Title** | Login Page Component |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a polished login page component with email/password form, validation, and error handling.

**Acceptance Criteria:**
- [ ] File created: `frontend/src/pages/Login.jsx`
- [ ] Email and password input fields
- [ ] Client-side validation
- [ ] Error message display
- [ ] Loading state during submission
- [ ] Styled with modern UI
- [ ] PR merged

---

### Task AG-33: Register Page Component

| Field | Value |
|-------|-------|
| **Task ID** | AG-33 |
| **Title** | Register Page Component |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 3 (Marva) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create a registration page component with email, password, and confirm password fields. Includes client-side validation.

**Acceptance Criteria:**
- [ ] File created: `frontend/src/pages/Register.jsx`
- [ ] Email, password, and confirm password fields
- [ ] Password strength validation
- [ ] Passwords must match
- [ ] Success message and redirect
- [ ] Styled consistently with login page
- [ ] PR merged

---

### Task AG-34: Auth API Service Module

| Field | Value |
|-------|-------|
| **Task ID** | AG-34 |
| **Title** | Auth API Service Module |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 1 (Shabas) |
| **Story Points** | 3 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create frontend API service module for authentication. Handles API calls to backend auth endpoints. Dev 1 learns frontend code!

**Acceptance Criteria:**
- [ ] File created: `frontend/src/services/api.js`
- [ ] Functions: register(), login(), logout()
- [ ] JWT token storage in localStorage
- [ ] Error handling for failed requests
- [ ] Integrates with backend endpoints
- [ ] PR merged

---

### Task AG-35: Protected Routes Setup

| Field | Value |
|-------|-------|
| **Task ID** | AG-35 |
| **Title** | Protected Routes Setup |
| **Type** | Task |
| **Epic** | Frontend & UI |
| **Assignee** | Dev 2 (Sinan) |
| **Story Points** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | High |

**Description:**
Create React Router protected route wrapper that redirects to login if user is not authenticated. Dev 2 learns frontend routing!

**Acceptance Criteria:**
- [ ] File created: `frontend/src/components/ProtectedRoute.jsx`  
- [ ] Checks for valid JWT token
- [ ] Redirects to /login if not authenticated
- [ ] Allows access if authenticated
- [ ] Applied to dashboard route
- [ ] PR merged

---

## � 9:30 AM - Start Coding

---

## 👩‍💻 Dev 3 (Marva): AG-32 - Login Page Component

### Step 1: Create Branch

```bash
git checkout marva-Dev && git pull origin main
git checkout -b feature/AG-32-login-page
```

**Jira:** Move AG-32 to "In Progress"

### Step 2: Install Dependencies

```bash
cd frontend
npm install react-router-dom
```

### Step 3: Create Login Page

Create file: `frontend/src/pages/Login.jsx`

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import './Auth.css';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    // Client-side validation
    if (!email || !password) {
      setError('Please fill in all fields');
      return;
    }

    if (!email.includes('@')) {
      setError('Please enter a valid email');
      return;
    }

    setLoading(true);

    try {
      // API call will be added in AG-34
      console.log('Login:', { email, password });
      
      // Temporary success simulation
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // Will navigate to dashboard once auth is connected
      alert('Login successful! (API integration coming in AG-34)');
      
    } catch (err) {
      setError(err.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <div className="auth-header">
          <h1>Welcome Back</h1>
          <p>Sign in to optimize your resume</p>
        </div>

        {error && (
          <div className="error-message">
            {error}
          </div>
        )}

        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email Address</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="you@example.com"
              disabled={loading}
              autoComplete="email"
            />
          </div>

          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Enter your password"
              disabled={loading}
              autoComplete="current-password"
            />
          </div>

          <button 
            type="submit" 
            className="auth-button" 
            disabled={loading}
          >
            {loading ? 'Signing in...' : 'Sign In'}
          </button>
        </form>

        <p className="auth-footer">
          Don't have an account? <Link to="/register">Sign up</Link>
        </p>
      </div>
    </div>
  );
}

export default Login;
```

### Step 4: Create Auth Styles

Create file: `frontend/src/pages/Auth.css`

```css
.auth-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.auth-card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  padding: 40px;
  width: 100%;
  max-width: 420px;
}

.auth-header {
  text-align: center;
  margin-bottom: 32px;
}

.auth-header h1 {
  font-size: 28px;
  font-weight: 700;
  color: #1a202c;
  margin: 0 0 8px 0;
}

.auth-header p {
  font-size: 16px;
  color: #718096;
  margin: 0;
}

.error-message {
  background: #fed7d7;
  border: 1px solid #fc8181;
  color: #c53030;
  padding: 12px 16px;
  border-radius: 8px;
  margin-bottom: 20px;
  font-size: 14px;
}

.success-message {
  background: #c6f6d5;
  border: 1px solid #68d391;
  color: #2f855a;
  padding: 12px 16px;
  border-radius: 8px;
  margin-bottom: 20px;
  font-size: 14px;
}

.auth-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  font-size: 14px;
  font-weight: 600;
  color: #2d3748;
}

.form-group input {
  padding: 12px 16px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.2s;
}

.form-group input:focus {
  outline: none;
  border-color: #667eea;
}

.form-group input:disabled {
  background: #f7fafc;
  cursor: not-allowed;
}

.form-hint {
  font-size: 13px;
  color: #718096;
  margin-top: 4px;
}

.auth-button {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 14px 24px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  margin-top: 8px;
}

.auth-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(102, 126, 234, 0.4);
}

.auth-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.auth-footer {
  text-align: center;
  margin-top: 24px;
  font-size: 14px;
  color: #718096;
}

.auth-footer a {
  color: #667eea;
  font-weight: 600;
  text-decoration: none;
}

.auth-footer a:hover {
  text-decoration: underline;
}
```

### Step 5: Update App Routes

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Login from './pages/Login';
import Register from './pages/Register';
import Dashboard from './pages/Dashboard';
import './App.css';

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/" element={<Login />} />
      </Routes>
    </Router>
  );
}

export default App;
```

### Step 6: Test Login Page

```bash
npm run dev
```

Open http://localhost:5173/login

**Test scenarios:**
- Try submitting empty form → Should show validation error
- Try invalid email format → Should show validation error
- Fill in both fields → Should show success alert

### Step 7: Commit and Push

```bash
git add frontend/src/pages/Login.jsx frontend/src/pages/Auth.css frontend/src/App.jsx
git commit -m "AG-32: Create login page component

- Email and password input fields
- Client-side validation
- Error message display
- Loading state
- Modern gradient UI design"

git push origin feature/AG-32-login-page
```

**Jira:** Move AG-32 to "Done"

---

## 👩‍💻 Dev 3 (Marva): AG-33 - Register Page Component

### Step 1: Create Branch

```bash
git checkout marva-Dev && git pull origin main
git checkout -b feature/AG-33-register-page
```

**Jira:** Move AG-33 to "In Progress"

### Step 2: Create Register Page

Create file: `frontend/src/pages/Register.jsx`

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import './Auth.css';

function Register() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();

  const validatePassword = (pwd) => {
    if (pwd.length < 8) {
      return 'Password must be at least 8 characters';
    }
    if (!/[A-Z]/.test(pwd)) {
      return 'Password must contain at least one uppercase letter';
    }
    if (!/[a-z]/.test(pwd)) {
      return 'Password must contain at least one lowercase letter';
    }
    if (!/[0-9]/.test(pwd)) {
      return 'Password must contain at least one number';
    }
    return null;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    setSuccess('');

    // Client-side validation
    if (!email || !password || !confirmPassword) {
      setError('Please fill in all fields');
      return;
    }

    if (!email.includes('@')) {
      setError('Please enter a valid email');
      return;
    }

    const passwordError = validatePassword(password);
    if (passwordError) {
      setError(passwordError);
      return;
    }

    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return;
    }

    setLoading(true);

    try {
      // API call will be added in AG-34
      console.log('Register:', { email, password });
      
      // Temporary success simulation
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      setSuccess('Account created successfully! Redirecting to login...');
      
      // Redirect to login after 2 seconds
      setTimeout(() => {
        navigate('/login');
      }, 2000);
      
    } catch (err) {
      setError(err.message || 'Registration failed');
      setLoading(false);
    }
  };

  const getPasswordStrength = () => {
    if (!password) return '';
    if (password.length < 6) return 'weak';
    if (password.length < 10 && /[A-Za-z0-9]/.test(password)) return 'medium';
    if (password.length >= 10 && /[A-Z]/.test(password) && /[a-z]/.test(password) && /[0-9]/.test(password)) {
      return 'strong';
    }
    return 'medium';
  };

  const passwordStrength = getPasswordStrength();

  return (
    <div className="auth-container">
      <div className="auth-card">
        <div className="auth-header">
          <h1>Create Account</h1>
          <p>Start optimizing your resume today</p>
        </div>

        {error && (
          <div className="error-message">
            {error}
          </div>
        )}

        {success && (
          <div className="success-message">
            {success}
          </div>
        )}

        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email Address</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="you@example.com"
              disabled={loading}
              autoComplete="email"
            />
          </div>

          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Create a strong password"
              disabled={loading}
              autoComplete="new-password"
            />
            {password && (
              <div className={`password-strength ${passwordStrength}`}>
                Strength: <strong>{passwordStrength}</strong>
              </div>
            )}
            <small className="form-hint">
             must be at least 8 characters with uppercase, lowercase, and number
            </small>
          </div>

          <div className="form-group">
            <label htmlFor="confirmPassword">Confirm Password</label>
            <input
              id="confirmPassword"
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              placeholder="Confirm your password"
              disabled={loading}
              autoComplete="new-password"
            />
          </div>

          <button 
            type="submit" 
            className="auth-button" 
            disabled={loading}
          >
            {loading ? 'Creating account...' : 'Create Account'}
          </button>
        </form>

        <p className="auth-footer">
          Already have an account? <Link to="/login">Sign in</Link>
        </p>
      </div>
    </div>
  );
}

export default Register;
```

### Step 3: Add Password Strength Styles

Update `frontend/src/pages/Auth.css` (add at the end):

```css
.password-strength {
  font-size: 13px;
  padding: 4px 8px;
  border-radius: 4px;
  margin-top: 4px;
}

.password-strength.weak {
  background: #fed7d7;
  color: #c53030;
}

.password-strength.medium {
  background: #feebc8;
  color: #c05621;
}

.password-strength.strong {
  background: #c6f6d5;
  color: #2f855a;
}
```

### Step 4: Update Routes

Update `frontend/src/App.jsx` to include the register route (should already be there from AG-32).

### Step 5: Test Register Page

```bash
npm run dev
```

Open http://localhost:5173/register

**Test scenarios:**
- Try submitting empty form → Validation error
- Enter weak password → See "weak" strength indicator
- Enter passwords that don't match → Error message
- Enter invalid email → Error message
- Fill all fields correctly → Success message and redirect

### Step 6: Commit and Push

```bash
git add frontend/src/pages/Register.jsx frontend/src/pages/Auth.css
git commit -m "AG-33: Create register page component

- Email, password, and confirm password fields
- Password strength validation and indicator
- Passwords must match validation
- Success message and auto-redirect
- Consistent styling with login page"

git push origin feature/AG-33-register-page
```

**Jira:** Move AG-33 to "Done"

---

## 👨‍💻 Dev 1 (Shabas): AG-34 - Auth API Service Module

### Step 1: Create Branch

```bash
git checkout shabas-Dev && git pull origin main
git checkout -b feature/AG-34-auth-api-service
```

**Jira:** Move AG-34 to "In Progress"

### Step 2: Create API Service

Create file: `frontend/src/services/api.js`

```javascript
/**
 * API Service Module
 * 
 * Handles all HTTP requests to the backend API.
 * Manages JWT token storage and authenticated requests.
 */

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

// ========== Token Management ==========

/**
 * Store JWT token in localStorage
 */
export function setToken(token) {
  localStorage.setItem('token', token);
}

/**
 * Get JWT token from localStorage
 */
export function getToken() {
  return localStorage.getItem('token');
}

/**
 * Remove JWT token from localStorage
 */
export function removeToken() {
  localStorage.removeItem('token');
}

/**
 * Check if user is authenticated
 */
export function isAuthenticated() {
  return !!getToken();
}

// ========== API Helper ==========

/**
 * Make authenticated API request
 */
async function apiRequest(endpoint, options = {}) {
  const url = `${API_URL}${endpoint}`;
  
  const headers = {
    'Content-Type': 'application/json',
    ...options.headers,
  };

  // Add Authorization header if token exists
  const token = getToken();
  if (token) {
    headers['Authorization'] = `Bearer ${token}`;
  }

  const config = {
    ...options,
    headers,
  };

  const response = await fetch(url, config);
  const data = await response.json();

  if (!response.ok) {
    // Handle error responses
    throw new Error(data.detail || data.error || 'Request failed');
  }

  return data;
}

// ========== Auth API Methods ==========

/**
 * Register a new user
 * @param {string} email - User email
 * @param {string} password - User password
 * @returns {Promise<{user_id: string, email: string, message: string}>}
 */
export async function register(email, password) {
  const data = await apiRequest('/api/auth/register', {
    method: 'POST',
    body: JSON.stringify({ email, password }),
  });

  return data;
}

/**
 * Login user and store token
 * @param {string} email - User email
 * @param {string} password - User password
 * @returns {Promise<{access_token: string, user: object}>}
 */
export async function login(email, password) {
  const data = await apiRequest('/api/auth/login', {
    method: 'POST',
    body: JSON.stringify({ email, password }),
  });

  // Store token
  if (data.access_token) {
    setToken(data.access_token);
  }

  return data;
}

/**
 * Logout user (clear token)
 */
export function logout() {
  removeToken();
}

/**
 * Get current user information
 * @returns {Promise<{id: string, email: string}>}
 */
export async function getCurrentUser() {
  return apiRequest('/api/user/me');
}

/**
 * Get user profile
 * @returns {Promise<object>}
 */
export async function getUserProfile() {
  return apiRequest('/api/user/profile');
}

// ========== Agent API Methods (Placeholders for Day 8) ==========

/**
 * Start resume optimization
 * @param {string} jobDescription - Job description text
 * @param {string} resume - Resume text
 * @returns {Promise<{run_id: string}>}
 */
export async function startOptimization(jobDescription, resume) {
  return apiRequest('/api/optimize', {
    method: 'POST',
    body: JSON.stringify({ job_description: jobDescription, resume }),
  });
}

/**
 * Get optimization result
 * @param {string} runId - Optimization run ID
 * @returns {Promise<object>}
 */
export async function getOptimizationResult(runId) {
  return apiRequest(`/api/runs/${runId}`);
}

/**
 * Get user's optimization history
 * @returns {Promise<Array>}
 */
export async function getOptimizationHistory() {
  return apiRequest('/api/runs');
}
```

### Step 3: Create Environment Variables

Create file: `frontend/.env`

```bash
VITE_API_URL=http://localhost:8000
```

Add to `frontend/.env.example`:

```bash
VITE_API_URL=http://localhost:8000
```

### Step 4: Update Login Page to Use API

Update `frontend/src/pages/Login.jsx`:

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { login } from '../services/api';
import './Auth.css';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    // Client-side validation
    if (!email || !password) {
      setError('Please fill in all fields');
      return;
    }

    if (!email.includes('@')) {
      setError('Please enter a valid email');
      return;
    }

    setLoading(true);

    try {
      // Call API
      const data = await login(email, password);
      
      console.log('Login successful:', data);
      
      // Redirect to dashboard
      navigate('/dashboard');
      
    } catch (err) {
      setError(err.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <div className="auth-header">
          <h1>Welcome Back</h1>
          <p>Sign in to optimize your resume</p>
        </div>

        {error && (
          <div className="error-message">
            {error}
          </div>
        )}

        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email Address</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="you@example.com"
              disabled={loading}
              autoComplete="email"
            />
          </div>

          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Enter your password"
              disabled={loading}
              autoComplete="current-password"
            />
          </div>

          <button 
            type="submit" 
            className="auth-button" 
            disabled={loading}
          >
            {loading ? 'Signing in...' : 'Sign In'}
          </button>
        </form>

        <p className="auth-footer">
          Don't have an account? <Link to="/register">Sign up</Link>
        </p>
      </div>
    </div>
  );
}

export default Login;
```

### Step 5: Update Register Page to Use API

Update `frontend/src/pages/Register.jsx`:

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { register } from '../services/api';
import './Auth.css';

function Register() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();

  const validatePassword = (pwd) => {
    if (pwd.length < 8) {
      return 'Password must be at least 8 characters';
    }
    if (!/[A-Z]/.test(pwd)) {
      return 'Password must contain at least one uppercase letter';
    }
    if (!/[a-z]/.test(pwd)) {
      return 'Password must contain at least one lowercase letter';
    }
    if (!/[0-9]/.test(pwd)) {
      return 'Password must contain at least one number';
    }
    return null;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    setSuccess('');

    // Client-side validation
    if (!email || !password || !confirmPassword) {
      setError('Please fill in all fields');
      return;
    }

    if (!email.includes('@')) {
      setError('Please enter a valid email');
      return;
    }

    const passwordError = validatePassword(password);
    if (passwordError) {
      setError(passwordError);
      return;
    }

    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return;
    }

    setLoading(true);

    try {
      // Call API
      await register(email, password);
      
      setSuccess('Account created successfully! Redirecting to login...');
      
      // Redirect to login after 2 seconds
      setTimeout(() => {
        navigate('/login');
      }, 2000);
      
    } catch (err) {
      setError(err.message || 'Registration failed');
      setLoading(false);
    }
  };

  const getPasswordStrength = () => {
    if (!password) return '';
    if (password.length < 6) return 'weak';
    if (password.length < 10 && /[A-Za-z0-9]/.test(password)) return 'medium';
    if (password.length >= 10 && /[A-Z]/.test(password) && /[a-z]/.test(password) && /[0-9]/.test(password)) {
      return 'strong';
    }
    return 'medium';
  };

  const passwordStrength = getPasswordStrength();

  return (
    <div className="auth-container">
      <div className="auth-card">
        <div className="auth-header">
          <h1>Create Account</h1>
          <p>Start optimizing your resume today</p>
        </div>

        {error && (
          <div className="error-message">
            {error}
          </div>
        )}

        {success && (
          <div className="success-message">
            {success}
          </div>
        )}

        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email Address</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="you@example.com"
              disabled={loading}
              autoComplete="email"
            />
          </div>

          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Create a strong password"
              disabled={loading}
              autoComplete="new-password"
            />
            {password && (
              <div className={`password-strength ${passwordStrength}`}>
                Strength: <strong>{passwordStrength}</strong>
              </div>
            )}
            <small className="form-hint">
              Must be at least 8 characters with uppercase, lowercase, and number
            </small>
          </div>

          <div className="form-group">
            <label htmlFor="confirmPassword">Confirm Password</label>
            <input
              id="confirmPassword"
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              placeholder="Confirm your password"
              disabled={loading}
              autoComplete="new-password"
            />
          </div>

          <button 
            type="submit" 
            className="auth-button" 
            disabled={loading}
          >
            {loading ? 'Creating account...' : 'Create Account'}
          </button>
        </form>

        <p className="auth-footer">
          Already have an account? <Link to="/login">Sign in</Link>
        </p>
      </div>
    </div>
  );
}

export default Register;
```

### Step 6: Test Full Auth Flow

```bash
# Terminal 1: Start backend
cd backend
uvicorn main:app --reload

# Terminal 2: Start frontend
cd frontend
npm run dev
```

**Test in browser:**
1. Go to http://localhost:5173/register
2. Register a new account
3. Should see success message and redirect to login
4. Login with same credentials
5. Should redirect to dashboard
6. Open DevTools → Application → LocalStorage → See token stored

### Step 7: Commit and Push

```bash
git add frontend/src/services/api.js frontend/src/pages/ frontend/.env frontend/.env.example
git commit -m "AG-34: Create auth API service module

- API service with register/login/logout functions
- JWT token storage in localStorage
- Error handling for failed requests
- Update Login page to call API
- Update Register page to call API
- Environment variables for API URL"

git push origin feature/AG-34-auth-api-service
```

**Jira:** Move AG-34 to "Done"

---

## 👨‍💻 Dev 2 (Sinan): AG-35 - Protected Routes Setup

### Step 1: Create Branch

```bash
git checkout sinan-Dev && git pull origin main
git checkout -b feature/AG-35-protected-routes
```

**Jira:** Move AG-35 to "In Progress"

### Step 2: Create ProtectedRoute Component

Create file: `frontend/src/components/ProtectedRoute.jsx`

```jsx
import { Navigate } from 'react-router-dom';
import { isAuthenticated } from '../services/api';

/**
 * Protected Route Wrapper
 * 
 * Redirects to login if user is not authenticated.
 * Use this to wrap any routes that require authentication.
 * 
 * Usage:
 *   <Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
 */
function ProtectedRoute({ children }) {
  if (!isAuthenticated()) {
    // Redirect to login if not authenticated
    return <Navigate to="/login" replace />;
  }

  // Render the protected component if authenticated
  return children;
}

export default ProtectedRoute;
```

### Step 3: Create Placeholder Dashboard

Create file: `frontend/src/pages/Dashboard.jsx`

```jsx
import { useNavigate } from 'react-router-dom';
import { logout, getCurrentUser } from '../services/api';
import { useState, useEffect } from 'react';
import './Dashboard.css';

function Dashboard() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const navigate = useNavigate();

  useEffect(() => {
    // Fetch current user info
    getCurrentUser()
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        console.error('Failed to fetch user:', err);
        // Token might be invalid, logout
        logout();
        navigate('/login');
      });
  }, [navigate]);

  const handleLogout = () => {
    logout();
    navigate('/login');
  };

  if (loading) {
    return (
      <div className="dashboard-container">
        <div className="loading">Loading...</div>
      </div>
    );
  }

  return (
    <div className="dashboard-container">
      <header className="dashboard-header">
        <h1>Resume Agent Dashboard</h1>
        <div className="header-actions">
          <span className="user-email">{user?.email}</span>
          <button onClick={handleLogout} className="logout-button">
            Logout
          </button>
        </div>
      </header>

      <main className="dashboard-main">
        <div className="welcome-card">
          <h2>Welcome to Resume Agent! 👋</h2>
          <p>You're successfully authenticated.</p>
          <p className="note">
            <strong>Coming in Day 8:</strong> Resume optimization form will be added here.
          </p>
        </div>

        <div className="feature-preview">
          <h3>What's Next:</h3>
          <ul>
            <li>✅ Authentication complete</li>
            <li>⏳ Upload resume and job description (Day 8)</li>
            <li>⏳ View optimization results (Day 9)</li>
            <li>⏳ Access optimization history (Sprint 2)</li>
          </ul>
        </div>
      </main>
    </div>
  );
}

export default Dashboard;
```

### Step 4: Create Dashboard Styles

Create file: `frontend/src/pages/Dashboard.css`

```css
.dashboard-container {
  min-height: 100vh;
  background: #f7fafc;
}

.dashboard-header {
  background: white;
  padding: 20px 40px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dashboard-header h1 {
  font-size: 24px;
  color: #1a202c;
  margin: 0;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 20px;
}

.user-email {
  color: #718096;
  font-size: 14px;
}

.logout-button {
  background: #e53e3e;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.logout-button:hover {
  background: #c53030;
}

.dashboard-main {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
}

.welcome-card {
  background: white;
  border-radius: 12px;
  padding: 40px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 30px;
  text-align: center;
}

.welcome-card h2 {
  font-size: 32px;
  color: #1a202c;
  margin: 0 0 16px 0;
}

.welcome-card p {
  font-size: 18px;
  color: #718096;
  margin: 8px 0;
}

.welcome-card .note {
  background: #edf2f7;
  padding: 16px;
  border-radius: 8px;
  margin-top: 24px;
  font-size: 16px;
}

.feature-preview {
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.feature-preview h3 {
  font-size: 20px;
  color: #1a202c;
  margin: 0 0 16px 0;
}

.feature-preview ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.feature-preview li {
  padding: 12px 0;
  font-size: 16px;
  color: #4a5568;
  border-bottom: 1px solid #e2e8f0;
}

.feature-preview li:last-child {
  border-bottom: none;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  font-size: 18px;
  color: #718096;
}
```

### Step 5: Update App Routes with Protection

Update `frontend/src/App.jsx`:

```jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import ProtectedRoute from './components/ProtectedRoute';
import Login from './pages/Login';
import Register from './pages/Register';
import Dashboard from './pages/Dashboard';
import './App.css';

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
        <Route 
          path="/dashboard" 
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          } 
        />
        <Route path="/" element={<Login />} />
      </Routes>
    </Router>
  );
}

export default App;
```

### Step 6: Test Protected Routes

```bash
# Make sure backend and frontend are running
cd frontend && npm run dev
```

**Test scenarios:**
1. **Without login:**
   - Navigate to http://localhost:5173/dashboard
   - Should redirect to /login

2. **After login:**
   - Login via /login page
   - Should redirect to /dashboard
   - Should see welcome message with your email
   - Dashboard should load successfully

3. **After logout:**
   - Click logout button
   - Should redirect to /login
   - Try accessing /dashboard again
   - Should redirect back to /login

4. **Manual token removal:**
   - Login and reach dashboard
   - Open DevTools → Application → LocalStorage
   - Delete the token key
   - Refresh the page
   - Should redirect to /login

### Step 7: Commit and Push

```bash
git add frontend/src/components/ProtectedRoute.jsx frontend/src/pages/Dashboard.jsx frontend/src/pages/Dashboard.css frontend/src/App.jsx
git commit -m "AG-35: Add protected routes setup

- ProtectedRoute wrapper component
- Checks for valid JWT token
- Redirects to login if not authenticated
- Dashboard page with user info and logout
- Applied protection to dashboard route"

git push origin feature/AG-35-protected-routes
```

**Jira:** Move AG-35 to "Done"

---


import os
from datetime import datetime, timedelta
from typing import Optional

from fastapi import APIRouter, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
from jose import jwt, JWTError

# Create router
router = APIRouter(prefix="/api/auth", tags=["Authentication"])

# Security setup
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

# Configuration
SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key-change-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_HOURS = 24

# In-memory user storage (will migrate to Supabase)
# Format: {email: {"id": str, "email": str, "password_hash": str}}
users_db = {}


# === Pydantic Models ===

class UserRegister(BaseModel):
    """Request model for user registration."""
    email: EmailStr
    password: str
    
    class Config:
        json_schema_extra = {
            "example": {
                "email": "user@example.com",
                "password": "SecurePassword123"
            }
        }


class UserLogin(BaseModel):
    """Request model for user login."""
    email: EmailStr
    password: str


class TokenResponse(BaseModel):
    """Response model for successful login."""
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    user_id: str


class UserResponse(BaseModel):
    """Response model for user data."""
    id: str
    email: str


class MessageResponse(BaseModel):
    """Generic message response."""
    message: str
    user_id: Optional[str] = None


# === Helper Functions ===

def hash_password(password: str) -> str:
    """Hash a password using bcrypt."""
    return pwd_context.hash(password)


def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify a password against its hash."""
    return pwd_context.verify(plain_password, hashed_password)


def create_access_token(data: dict, expires_delta: timedelta = None) -> str:
    """Create a JWT access token."""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(hours=ACCESS_TOKEN_EXPIRE_HOURS)
    
    to_encode.update({"exp": expire})
    
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def decode_token(token: str) -> dict:
    """Decode and verify a JWT token."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None


# === API Endpoints ===

@router.post("/register", response_model=MessageResponse, status_code=status.HTTP_201_CREATED)
def register(user: UserRegister):
    """
    Register a new user.
    
    - **email**: Valid email address
    - **password**: Password (min 8 characters recommended)
    """
    # Check if email already exists
    if user.email.lower() in users_db:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # Validate password strength (basic)
    if len(user.password) < 6:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Password must be at least 6 characters"
        )
    
    # Create user
    import uuid
    user_id = str(uuid.uuid4())
    
    users_db[user.email.lower()] = {
        "id": user_id,
        "email": user.email.lower(),
        "password_hash": hash_password(user.password),
        "created_at": datetime.utcnow().isoformat()
    }
    
    return MessageResponse(
        message="User registered successfully",
        user_id=user_id
    )


@router.post("/login", response_model=TokenResponse)
def login(user: UserLogin):
    """
    Login and receive JWT token.
    
    - **email**: Registered email address
    - **password**: User's password
    """
    # Find user
    stored_user = users_db.get(user.email.lower())
    
    if not stored_user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Verify password
    if not verify_password(user.password, stored_user["password_hash"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid email or password"
        )
    
    # Create token
    access_token = create_access_token(
        data={
            "sub": stored_user["id"],
            "email": stored_user["email"]
        }
    )
    
    return TokenResponse(
        access_token=access_token,
        token_type="bearer",
        expires_in=ACCESS_TOKEN_EXPIRE_HOURS * 3600,
        user_id=stored_user["id"]
    )


@router.get("/me", response_model=UserResponse)
def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """
    Get current authenticated user's information.
    
    Requires Bearer token in Authorization header.
    """
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token"
        )
    
    user_id = payload.get("sub")
    email = payload.get("email")
    
    # Verify user still exists
    stored_user = users_db.get(email)
    if not stored_user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found"
        )
    
    return UserResponse(id=user_id, email=email)


# === Dependency for Protected Routes ===

def get_current_user_id(credentials: HTTPAuthorizationCredentials = Depends(security)) -> str:
    """
    Dependency to get current user ID from token.
    
    Usage:
        @router.get("/protected")
        def protected_route(user_id: str = Depends(get_current_user_id)):
            ...
    """
    token = credentials.credentials
    payload = decode_token(token)
    
    if not payload:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token"
        )
    
    return payload.get("sub")
```

### Step 3: Update Main App

Update file: `backend/main.py` - add these lines:

```python
# Add at the imports section
from api.auth import router as auth_router

# Add after the app definition (before the existing routes)
app.include_router(auth_router)
```

### Step 4: Test the Endpoints

```bash
cd backend
uvicorn main:app --reload

# In another terminal:

# 1. Test registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "password123"}'

# Expected: {"message":"User registered successfully","user_id":"..."}

# 2. Test login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "password123"}'

# Expected: {"access_token":"eyJ...","token_type":"bearer",...}

# 3. Test invalid login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "wrongpassword"}'

# Expected: {"detail":"Invalid email or password"}

# 4. Test duplicate registration
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "test@test.com", "password": "password123"}'

# Expected: {"detail":"Email already registered"}

# 5. Test /me endpoint (replace TOKEN with actual token from login)
curl http://localhost:8000/api/auth/me \
  -H "Authorization: Bearer TOKEN"

# Expected: {"id":"...","email":"test@test.com"}
```

### Step 5: Commit and PR

```bash
git add backend/api/ backend/main.py
git commit -m "AG-25: Create auth endpoints

- POST /api/auth/register - Create new user
- POST /api/auth/login - Return JWT token
- GET /api/auth/me - Get current user
- Password hashing with bcrypt
- JWT with 24h expiry"

git push origin feature/AG-25-auth-endpoints
```

**Jira:** Move AG-25 to "Done"

---

## 👨‍💻 Dev 3 (Marva): AG-29 - Frontend API Connection

### Step 1: Create Branch

```bash
git checkout dev-marva && git pull origin main
git checkout -b feature/AG-29-api-connection
```

**Jira:** Move AG-29 to "In Progress"

### Step 2: Create API Service

Create file: `frontend/src/services/api.js`

```javascript
/**
 * API Service
 * 
 * Handles all HTTP requests to the backend API.
 */

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

/**
 * Register a new user
 */
export async function register(email, password) {
  const response = await fetch(`${API_URL}/api/auth/register`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ email, password }),
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.detail || 'Registration failed');
  }

  return data;
}

/**
 * Login user and store token
 */
export async function login(email, password) {
  const response = await fetch(`${API_URL}/api/auth/login`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ email, password }),
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.detail || 'Login failed');
  }

  // Store token and user info
  localStorage.setItem('token', data.access_token);
  localStorage.setItem('userId', data.user_id);

  return data;
}

/**
 * Logout user
 */
export function logout() {
  localStorage.removeItem('token');
  localStorage.removeItem('userId');
}

/**
 * Check if user is logged in
 */
export function isLoggedIn() {
  return !!localStorage.getItem('token');
}

/**
 * Get stored token
 */
export function getToken() {
  return localStorage.getItem('token');
}

/**
 * Get current user info
 */
export async function getCurrentUser() {
  const token = getToken();
  
  if (!token) {
    throw new Error('Not logged in');
  }

  const response = await fetch(`${API_URL}/api/auth/me`, {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });

  if (!response.ok) {
    throw new Error('Failed to get user info');
  }

  return response.json();
}
```

### Step 3: Update Login Page

Update file: `frontend/src/pages/Login.jsx`

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { login } from '../services/api';
import './Auth.css';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  const navigate = useNavigate();

  const validateForm = () => {
    setError('');
    
    if (!email.trim()) {
      setError('Email is required');
      return false;
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Please enter a valid email address');
      return false;
    }
    
    if (!password) {
      setError('Password is required');
      return false;
    }
    
    return true;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      await login(email, password);
      navigate('/dashboard');
    } catch (err) {
      setError(err.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>Welcome Back</h1>
        <p className="auth-subtitle">Sign in to continue</p>
        
        {error && <div className="error-message">{error}</div>}
        
        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              disabled={loading}
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Enter your password"
              disabled={loading}
            />
          </div>
          
          <button type="submit" className="auth-button" disabled={loading}>
            {loading ? 'Signing in...' : 'Sign In'}
          </button>
        </form>
        
        <p className="auth-footer">
          Don't have an account? <Link to="/register">Create one</Link>
        </p>
      </div>
    </div>
  );
}

export default Login;
```

### Step 4: Update Register Page

Update file: `frontend/src/pages/Register.jsx`

```jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { register } from '../services/api';
import './Auth.css';

function Register() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);
  
  const navigate = useNavigate();

  const validateForm = () => {
    setError('');
    
    if (!email.trim()) {
      setError('Email is required');
      return false;
    }
    
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      setError('Please enter a valid email address');
      return false;
    }
    
    if (!password) {
      setError('Password is required');
      return false;
    }
    
    if (password.length < 8) {
      setError('Password must be at least 8 characters');
      return false;
    }
    
    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return false;
    }
    
    return true;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setLoading(true);
    setError('');
    
    try {
      await register(email, password);
      setSuccess(true);
      
      // Redirect to login after 2 seconds
      setTimeout(() => {
        navigate('/login');
      }, 2000);
    } catch (err) {
      setError(err.message || 'Registration failed');
    } finally {
      setLoading(false);
    }
  };

  if (success) {
    return (
      <div className="auth-container">
        <div className="auth-card">
          <h1>✓ Account Created!</h1>
          <p>Redirecting to login...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="auth-container">
      <div className="auth-card">
        <h1>Create Account</h1>
        <p className="auth-subtitle">Get started with Resume Agent</p>
        
        {error && <div className="error-message">{error}</div>}
        
        <form onSubmit={handleSubmit} className="auth-form">
          <div className="form-group">
            <label htmlFor="email">Email</label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              disabled={loading}
            />
          </div>
          
          <div className="form-group">
            <label htmlFor="password">Password</label>
            <input
              id="password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Create a password"
              disabled={loading}
            />
            <small className="form-hint">At least 8 characters</small>
          </div>
          
          <div className="form-group">
            <label htmlFor="confirmPassword">Confirm Password</label>
            <input
              id="confirmPassword"
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              placeholder="Confirm your password"
              disabled={loading}
            />
          </div>
          
          <button type="submit" className="auth-button" disabled={loading}>
            {loading ? 'Creating account...' : 'Create Account'}
          </button>
        </form>
        
        <p className="auth-footer">
          Already have an account? <Link to="/login">Sign in</Link>
        </p>
      </div>
    </div>
  );
}

export default Register;
```

### Step 5: Test the Full Flow

```bash
# Terminal 1: Backend
cd backend && uvicorn main:app --reload

# Terminal 2: Frontend
cd frontend && npm run dev
```

### Step 6: Commit and PR

```bash
git add frontend/
git commit -m "AG-29: Connect frontend to auth API

- Create API service with login/register functions
- Store JWT token in localStorage
- Update Login page to call API
- Update Register page to call API
- Redirect on success"

git push origin feature/AG-29-api-connection
```

**Jira:** Move AG-29 to "Done"

---

## ✅ Day 7: Manual Verification

### Backend Check (from Day 6)

```bash
# Verify backend is running
curl http://localhost:8000/health
```

### Frontend Tests

**Start both servers:**
```bash
# Terminal 1: Backend
cd backend && uvicorn main:app --reload

# Terminal 2: Frontend  
cd frontend && npm run dev
```

**Test Complete Auth Flow:**

1. **Register:**
   - Go to http://localhost:5173/register
   - Fill in: email@test.com, Password123, Password123
   - Click "Create Account"
   - ✓ Should see success message
   - ✓ Should redirect to /login after 2 seconds

2. **Login:**
   - Enter same credentials
   - Click "Sign In"
   - ✓ Should redirect to /dashboard
   - ✓ Should see welcome message with email

3. **Protected Route:**
   - Open DevTools → Application → LocalStorage
   - ✓ Should see `token` key with JWT value
   - Try to access /dashboard directly
   - ✓ Should work without redirecting

4. **Logout:**
   - Click logout button on dashboard
   - ✓ Should redirect to /login
   - ✓ Token should be removed from localStorage

5. **Protection Works:**
   - After logout, try to access http://localhost:5173/dashboard
   - ✓ Should redirect to /login immediately

### What Must Work

| Test | Expected |
|------|----------|
| Register new user | Success message, redirect to login |
| Register duplicate email | Error message from API |
| Login with valid credentials | Redirect to dashboard, token stored |
| Login with invalid credentials | Error message |
| Access dashboard with token | Dashboard loads with user info |
| Access dashboard without token | Redirect to login |
| Logout | Token removed, redirect to login |
| Password strength indicator | Shows weak/medium/strong |

### If Something Fails

Common issues:
1. **CORS errors** → Check backend CORS middleware allows http://localhost:5173
2. **Can't connect to API** → Verify backend running on :8000, frontend .env has correct URL
3. **Token not stored** → Check browser console for errors, verify api.js setToken() called
4. **Can't access dashboard** → Check ProtectedRoute is wrapping Dashboard in App.jsx
5. **401 Unauthorized** → Token might be expired, try logging in again

---

## 📁 Folder Structure After Day 7

```
frontend/src/
├── components/
│   └── ProtectedRoute.jsx  ✓ NEW - Auth guard
│
├── pages/
│   ├── Login.jsx           ✓ NEW - Login page
│   ├── Register.jsx        ✓ NEW - Register page
│   ├── Dashboard.jsx       ✓ NEW - Protected dashboard
│   ├── Auth.css            ✓ NEW - Auth styles
│   └── Dashboard.css       ✓ NEW - Dashboard styles
│
├── services/
│   └── api.js              ✓ NEW - API client
│
├── App.jsx                 ✓ Updated - Routes
└── .env                    ✓ NEW - API URL config
```

---

## 🔄 Evening Standup

**What did we accomplish today?**

| Dev | Task | Status |
|-----|------|--------|
| Marva | AG-32: Login Page Component | ✅ Complete |
| Marva | AG-33: Register Page Component | ✅ Complete |
| Shabas | AG-34: Auth API Service Module | ✅ Complete |
| Sinan | AG-35: Protected Routes Setup | ✅ Complete |

**Key achievements:**
- ✅ Complete auth UI (login + register pages)
- ✅ API service module with token management
- ✅ Protected routes working
- ✅ Full auth flow: register → login → dashboard → logout
- ✅ Cross-training: Shabas learned frontend, Sinan learned React Router!

**Blockers:** None

**Tomorrow (Day 8):**
- AG-36: POST /optimize endpoint (connect agent to API)
- AG-37: GET /runs/{id} endpoint
- AG-38: GET /runs history endpoint
- Start building the optimization form in frontend

**Notes:**
- Auth flow is complete and secure
- Ready to connect the agent to the API
- Team is comfortable with full-stack development

---

## 📊 Sprint 1 Progress

| Day | Focus | Tasks | Status |
|-----|-------|-------|--------|
| 1 | Setup | AG-7 to AG-14 | ✅ Complete |
| 2 | First Node | AG-15 to AG-17 | ✅ Complete |
| 3 | Three Nodes | AG-18 to AG-20 | ✅ Complete |
| 4 | Two Nodes | AG-21 to AG-23 | ✅ Complete |
| 5 | Workflow | AG-24 to AG-27 | ✅ Complete |
| 6 | Auth Backend | AG-28 to AG-31 | ✅ Complete |
| **7** | **Auth Frontend** | **AG-32 to AG-35** | **✅ Complete** |
| 8 | Agent API | AG-36 to AG-38 | 📅 Tomorrow |

**Sprint 1 Progress:** 35/45 tasks (78%) ✅

---

**← [Day 6](./day-06.md)** | **[Day 8: Agent API Endpoints →](./day-08.md)**
