# 🎯 Team Task Manager - Interview Walkthrough Guide

**Project Name:** Team Task Manager (Ethara AI Project)  
**Interview Duration:** 30-45 minutes (adjustable)  
**Created:** May 5, 2026  
**Repository:** https://github.com/Sibyraj1412/Ethara-AI-Project

---

## 📋 Quick Summary

**What is this project?**
A full-stack Team Task Manager web application that enables teams to create projects, assign tasks, track progress with role-based access control.

**Tech Stack:**
- Backend: Node.js + Express + MongoDB + JWT
- Frontend: React 18 + Vite + Axios
- Deployment: Railway

**Key Features:**
✅ User authentication with JWT (email/password, 7-day token)
✅ Role-based access: Admin (full control) vs Member (limited)
✅ Project management: create, add members
✅ Task management: create, assign, prioritize (Low/Medium/High)
✅ Kanban dashboard: Todo → InProgress → Completed
✅ Overdue tracking: red badges on overdue tasks
✅ Real-time stats: total tasks, in progress, completed, overdue

---

## 🚀 Elevator Pitch (30 seconds)

This is a full-stack Team Task Manager application built with Node.js, Express, React, and MongoDB. It enables teams to create projects, assign tasks, and track progress in real-time with role-based access control. Key features include JWT authentication, a Kanban-style dashboard, task prioritization, and overdue tracking. It's production-ready and deployed on Railway.

---

## ��️ Architecture Overview

\\\
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT (React + Vite)                     │
│  Login/Signup → Dashboard → Projects → Tasks → Kanban Board  │
│                (Axios HTTP Client with JWT)                  │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTPS / REST API
┌──────────────────────┴──────────────────────────────────────┐
│                 SERVER (Node.js + Express)                   │
│  /api/auth   /api/projects   /api/tasks                      │
│  + JWT Middleware  + Admin-Only Middleware                   │
└──────────────────────┬──────────────────────────────────────┘
                       │ Mongoose ODM
┌──────────────────────┴──────────────────────────────────────┐
│               DATABASE (MongoDB - Atlas)                     │
│  Collections: Users │ Projects │ Tasks                       │
│  Indexed on: userId, projectId for fast queries             │
└──────────────────────────────────────────────────────────────┘
\\\

---

## 🔐 Authentication Flow (5 min explanation)

### Signup Process
1. User fills form: name, email, password, role
2. POST /api/auth/signup with credentials
3. Backend validates input (required fields, email format)
4. Check if email already exists (unique index in MongoDB)
5. Hash password with bcryptjs (salt rounds = 10)
   - This is irreversible; even if DB is compromised, passwords can't be read
6. Create User document in MongoDB
7. Generate JWT token: jwt.sign({ id, email, role }, JWT_SECRET, expires: 7d)
8. Return token + user info to frontend
9. Frontend stores token in localStorage
10. Page refreshes → reads token from localStorage → shows Dashboard (no new login needed)

### Login Process
1. User enters email + password
2. POST /api/auth/login
3. Find user by email in database
4. Compare entered password with stored hash using bcrypt.compare()
   - bcrypt.compare() is one-way; it hashes the entered password and compares hashes
5. If passwords don't match: return 401 "Invalid credentials"
6. If match: generate JWT token (same as signup)
7. Frontend stores token, redirects to Dashboard

### Token Usage
- Stored in localStorage as 	oken
- Axios interceptor automatically adds token to every API request:
  `javascript
  Authorization: Bearer <token>
  `
- Backend auth middleware verifies token on protected routes:
  1. Extract token from Authorization header
  2. jwt.verify(token, JWT_SECRET) checks signature and expiration
  3. If valid: decode and attach user info to req.user
  4. If invalid: return 401 Unauthorized

### Security Points
✅ Passwords never stored plain text
✅ JWT_SECRET stored in .env (never in git)
✅ Tokens expire after 7 days
✅ bcryptjs uses 10 salt rounds (industry standard)
✅ CORS configured to accept only known origins

---

## 📊 Database Schema (Show this during demo)

### Users Collection
\\\javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (unique, required),
  password: String (hashed with bcrypt),
  role: String (enum: ['Admin', 'Member']),
  createdAt: Date (auto)
}
\\\

### Projects Collection
\\\javascript
{
  _id: ObjectId,
  name: String (required),
  description: String,
  owner: ObjectId (ref: User),  // Creator of project
  members: [
    { userId: ObjectId (ref: User), role: String }  // Team members
  ],
  status: String (enum: ['Active', 'Completed', 'OnHold']),
  createdAt: Date (auto),
  updatedAt: Date (auto)
}
\\\

### Tasks Collection
\\\javascript
{
  _id: ObjectId,
  title: String (required),
  description: String,
  project: ObjectId (ref: Project),
  assignedTo: ObjectId (ref: User),  // Who it's assigned to
  status: String (enum: ['Todo', 'InProgress', 'Completed']),
  priority: String (enum: ['Low', 'Medium', 'High']),
  dueDate: Date,
  isOverdue: Boolean,  // Auto-calculated: dueDate < today AND status !== 'Completed'
  createdBy: ObjectId (ref: User),  // Who created the task
  createdAt: Date (auto),
  updatedAt: Date (auto)
}
\\\

---

## 🛠️ Tech Stack & Why

### Backend
- **Node.js + Express:** Fast, non-blocking I/O, great for real-time APIs
- **MongoDB + Mongoose:** NoSQL flexibility, enforces schema with Mongoose ODM
- **JWT (jsonwebtoken):** Stateless auth, scales to microservices
- **bcryptjs:** Secure password hashing, irreversible one-way function
- **CORS:** Handles cross-origin requests from React frontend safely
- **dotenv:** Manages environment variables (secrets not in git)

### Frontend
- **React 18:** Component-based UI, efficient state management
- **Vite:** Fast build tool, instant hot reload during development
- **Axios:** Promise-based HTTP client, interceptors for automatic JWT injection
- **CSS3:** Native styling, Flexbox for responsive Kanban layout

---

## �� Live Demo Script (5-10 minutes)

### Part 1: Signup (1 min)
1. Show login page at http://localhost:3000
2. Click "Sign Up"
3. Fill form:
   - Name: Test Admin
   - Email: admin@test.com
   - Password: Test123456!
   - Role: Admin
4. Click Sign Up
5. Explain: Password is hashed, JWT generated, stored in localStorage, redirected to Dashboard

### Part 2: Create Project (1 min)
1. Dashboard shows: "Welcome, Test Admin"
2. Click "New Project" button
3. Fill: Name = "Q1 Marketing", Description = "Spring marketing initiatives"
4. Click Create
5. Project appears in list
6. Explain: Backend assigns current user as owner with Admin role

### Part 3: Create Tasks (2 min)
1. Click on project to view tasks
2. Click "New Task"
3. Create 3 tasks:
   - Task 1: "Design Banners" | Priority: High (red) | Due: May 15 | Assign to self
   - Task 2: "Social Posts" | Priority: Medium (yellow) | Due: May 10
   - Task 3: "Research" | Priority: Low (green) | Due: May 1 (PAST DATE)
4. Click Save for each
5. Explain: Priority color-coded, due date tracked, assigned to users

### Part 4: Kanban Board (2 min)
1. Show three columns: Todo | InProgress | Completed
2. All 3 tasks in Todo column
3. Drag "Design Banners" → InProgress
4. Explain: Drag-drop updates status, real-time update
5. Drag "Research" → Completed (notice isOverdue on it)
6. Explain: Overdue badge shows on "Social Posts" (due date = May 10 = past)
7. Show: Dashboard stats update (Total: 3, In Progress: 1, Completed: 1, Overdue: 1)

### Part 5: Show Key Code (1-2 min)
Open backend/src/routes/authRoutes.js and explain:
- Password hashing in pre-save hook
- JWT generation with 7-day expiration
- Error handling (400, 409, 500 status codes)

---

## 💻 Setup & Run (PowerShell Commands)

\\\powershell
# Backend (Terminal 1)
cd C:\Users\sibyr\Ethara-AI-Project\backend
npm install
Copy-Item .env.example .env
# Edit .env: PORT=5000, MONGODB_URI=mongodb://..., JWT_SECRET=secret123
npm run dev

# Frontend (Terminal 2)
cd C:\Users\sibyr\Ethara-AI-Project\frontend
npm install
Copy-Item .env.example .env
# Edit .env: VITE_API_URL=http://localhost:5000/api
npm run dev

# Open browser: http://localhost:3000
\\\

---

## 🔑 Key Files to Reference

| File | Purpose |
|------|---------|
| backend/src/models/User.js | User schema + password hashing pre-hook |
| backend/src/models/Project.js | Project schema with members array |
| backend/src/models/Task.js | Task schema with status, priority, dueDate |
| backend/src/middleware/auth.js | JWT verification + admin role check |
| backend/src/routes/authRoutes.js | Signup/Login endpoints |
| backend/src/routes/projectRoutes.js | Project CRUD endpoints |
| backend/src/routes/taskRoutes.js | Task CRUD endpoints |
| frontend/src/App.jsx | SPA routing (Login → Dashboard) |
| frontend/src/components/Dashboard.jsx | Main app, Kanban board, project list |
| frontend/src/services/api.js | Axios config + JWT interceptor |

---

## ❓ Common Interview Questions

### Q1: How does authentication work?
A: User signs up → password hashed with bcryptjs → JWT token generated → stored in localStorage → Axios interceptor adds token to all requests → backend verifies token on protected routes.

### Q2: What's the difference between 401 and 403?
A: 401 = Unauthorized (no valid token), 403 = Forbidden (token valid but insufficient permissions).

### Q3: Why use MongoDB instead of PostgreSQL?
A: MongoDB is NoSQL with flexible schema, free tier, integrates well with Node.js. PostgreSQL would require schema migrations.

### Q4: How do you prevent password hacking?
A: Passwords are hashed irreversibly with bcryptjs (10 salt rounds). Even if DB is stolen, passwords can't be recovered.

### Q5: What would you improve?
A: Add unit tests, implement websockets for real-time collaboration, add search/filtering, implement rate limiting, add email notifications.

### Q6: How does role-based access work?
A: Admin can create projects/tasks. Members can view projects/tasks but can't create new ones. Admin middleware checks req.user.role on protected routes.

---

## 📝 Key Code Snippets

### Password Hashing (Pre-Save Hook)
\\\javascript
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});
\\\

### JWT Generation
\\\javascript
const generateToken = (user) => {
  return jwt.sign(
    { id: user._id, email: user.email, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '7d' }
  );
};
\\\

### Auth Middleware
\\\javascript
const auth = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'No token' });
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Invalid token' });
  }
};
\\\

### Axios Interceptor
\\\javascript
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = \Bearer \\;
  }
  return config;
});
\\\

### Create Task (Validation + Authorization)
\\\javascript
router.post('/tasks', auth, async (req, res) => {
  if (!req.body.title || !req.body.projectId) {
    return res.status(400).json({ message: 'Required fields missing' });
  }
  
  const project = await Project.findById(req.body.projectId);
  const isAdmin = project.owner.toString() === req.user.id;
  if (!isAdmin) {
    return res.status(403).json({ message: 'Admin access required' });
  }
  
  const task = new Task({ ...req.body, createdBy: req.user.id });
  await task.save();
  res.status(201).json({ message: 'Task created', data: task });
});
\\\

---

## ✅ Interview Checklist

- [ ] Know 30-second pitch
- [ ] Understand architecture (3 layers: frontend, backend, DB)
- [ ] Explain authentication flow (password hashing, JWT, token storage)
- [ ] Explain role-based access (admin vs member)
- [ ] Demo the app (signup, create project, create tasks, drag-drop)
- [ ] Show key code files (User.js, auth.js, authRoutes.js, Dashboard.jsx)
- [ ] Explain why tech choices (Node.js, MongoDB, React)
- [ ] Know API endpoints by heart (/auth/signup, /projects, /tasks)
- [ ] Mention security (bcryptjs, JWT_SECRET, CORS)
- [ ] Be ready to answer scaling questions (caching, load balancing, microservices)

---

**Good luck! You've built a solid, production-ready application. Present with confidence! 🚀**
