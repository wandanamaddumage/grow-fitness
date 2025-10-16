# Cursor Plan Mode Prompt — Grow Fitness Admin Panel (MVP)

You are an expert full-stack engineer. Build the **Grow Fitness Admin Panel (MVP)** using:

- **Backend:** NestJS, MongoDB (Mongoose), TypeScript  
- **Frontend:** Next.js (App Router), React 19, TypeScript, TailwindCSS, **shadcn/ui** (or a proper admin template using shadcn primitives)  
- **Email:** Nodemailer (SMTP dev transport)  
- **PDF/CSV:** `pdf-lib` or `puppeteer` for server-side PDF, `papaparse` or custom for CSV  
- **Jobs/Timers:** use `@nestjs/schedule` for nightly milestone jobs + daily digest; polling for notifications (no websockets yet)  
- **Auth:** Single **Admin** role. Session or JWT-based authentication.  
- **Currency:** **LKR**  
- **Calendar:** Tailwind-based calendar component.  
- **No payment gateway, no file uploads, hardcoded email templates.**

---

## Scope (Admin Panel Only)

### 1. Dashboard
- KPIs: Today’s sessions, pending reschedule/cancel requests, upcoming week’s sessions, milestone awards, invoices summary (LKR).  
- Activity feed: 20 recent CRM events.  
- Filters by date range and coach.

### 2. Users & Profiles
- CRUD: Parents (multi-child), Children, Coaches.  
- **Impersonate** feature (view-as Parent/Coach in read-only mode).  
- List, edit, and delete from admin.

### 3. Locations
- CRUD for session locations.  
- Dropdown field for locations in session creation.

### 4. Sessions
- Two types: **Individual** (1 child) and **Group** (N children).  
- Admin creates sessions by choosing coach, children, location, date/time, and capacity (for groups).  
- Conflict detection for overlapping coach or child sessions.  
- Session statuses: booked, canceled, completed.

### 5. Requests (Reschedule/Cancel)
- Requests created by parents/coaches (simulate via seed).  
- Admin queue: Approve/Reject with reason.  
- **Approval flow:**  
  - Approve Reschedule → pick new slot → update session → requeue reminders.  
  - Approve Cancel → update session → purge reminders.  
- **Late requests (<12h)** → mark as late, add fee note to CRM.

### 6. Reminders & Digests
- Reminder emails at **T-24h** and **T-1h** (skip T-24h if session <24h).  
- Daily digest email at 06:00 (local) for admin + coaches: today’s sessions and pending items.

### 7. Milestones (Digital Only)
- Rules-based milestones (by session count or BMI delta).  
- Nightly cron job awards badge/certificate (PDF or static asset).  
- Logs CRM entry + sends hardcoded congratulatory email.

### 8. Invoices (Manual)
- CRUD invoices: parent, amount (LKR), status (Paid/Unpaid), paid date, method (cash/bank/other).  
- Finance summary + exports (CSV/PDF).

### 9. Resources & Quizzes
- CRUD for parenting/fitness resources.  
- CRUD for quiz definitions and view submissions.

### 10. CRM Timeline & Audit Logs
- Log major actions: session create/update/delete, approve/reject requests, invoice changes, milestone awards.  
- CRM timeline shows detailed event stream with actor + timestamp.

### 11. Reports
- Weekly and monthly summaries:  
  - Attendance (scheduled vs completed vs canceled/late).  
  - Coach performance (session counts & earnings).  
  - Child activity (sessions, milestones).  
  - Finance summary (invoice totals in LKR).  
- CSV and PDF downloads.

### 12. Calendar
- Day/Week views for sessions.  
- Filter by coach, child, type.  
- Tailwind calendar component (user-specific view).

### 13. Notifications
- Admin panel polls `/notifications` every 30s for updates.

---

## Data Model (MongoDB/Mongoose)

Define schemas:
- **User:** `{ _id, role: 'admin'|'coach'|'parent', email, name, phone, passwordHash, createdAt }`
- **ParentProfile:** `{ userId, children: ObjectId[] }`
- **Child:** `{ parentId, name, birthDate, goals: string[] }`
- **CoachProfile:** `{ userId, skills: string[], availability: any[], earningsDerived?: number }`
- **Location:** `{ label }`
- **Session:** `{ type, coachId, childIds: ObjectId[], locationId, startAt, endAt, status }`
- **Request:** `{ type, sessionId, requesterId, reason, isLate, status, adminNote, decidedAt }`
- **ProgressLog:** `{ sessionId, childId, notes, milestones: string[] }`
- **MilestoneRule:** `{ name, conditionJSON, rewardType, isActive }`
- **MilestoneAward:** `{ childId, ruleId, awardedAt, artifactUrl }`
- **Invoice:** `{ parentId, amountLKR, status, paidDate, paidMethod }`
- **Resource:** `{ title, category, tags, contentRef }`
- **QuizResult:** `{ ownerType, ownerId, type, score }`
- **Notification:** `{ userId, type, payload, readAt }`
- **CRMEvent:** `{ actorId, subjectId, kind, payload, createdAt }`
- **AuditLog:** `{ actorId, action, targetType, targetId, meta, createdAt }`

Indexes: sessions (coach/date), requests (status), invoices (status/date).

---

## Backend (NestJS)
- Auth (Admin only)
- CRUD APIs for all modules (protected by AdminGuard)
- Scheduled tasks: reminders, daily digests, milestone awards
- CSV/PDF generation endpoints
- Nodemailer service for emails (hardcoded templates)

### Folder Layout
```
/api/src/modules/
  auth/
  users/
  parents/
  children/
  coaches/
  locations/
  sessions/
  requests/
  progress/
  milestones/
  invoices/
  resources/
  quizzes/
  crm/
  audit/
  notifications/
  reports/
  mailer/
  scheduler/
```

---

## Frontend (Next.js + shadcn/ui)

### Structure
```
/admin/app/
  /dashboard
  /users/{parents,children,coaches}
  /locations
  /sessions
  /requests
  /invoices
  /resources
  /quizzes
  /crm
  /audit
  /reports
  /login
```

### Pages
- **Dashboard:** KPI cards, feed, polling notifications.  
- **Users:** Tables + modals for CRUD.  
- **Sessions:** Calendar + table views, creation modal with conflict detection.  
- **Requests:** Approve/Reject modals.  
- **Invoices:** Mark paid, CSV/PDF export.  
- **Reports:** Weekly/monthly dashboards with download buttons.  
- **CRM/Audit:** Timeline list views.

### Components
- `DataTable`, `DialogForm`, `Calendar`, `KpiCard`, `Timeline`, `CsvExportBtn`, `PdfExportBtn`.

---

## Acceptance Criteria
- Create/approve/cancel/reschedule sessions with full flow and reminders.  
- All CRUD operations functional for users, sessions, invoices, requests, milestones, resources.  
- Reports downloadable (CSV/PDF).  
- Notifications poll every 30s.  
- CRM and Audit entries for major actions.  
- All values in **LKR**.  
- Email reminders/digests/milestones sent correctly (dev logs okay).

---

## Project Setup

**Environment:**
```
MONGO_URI=
JWT_SECRET=
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=
APP_BASE_URL=http://localhost:3000
CRON_TZ=Asia/Colombo
```

**Seed:**
- 1 Admin, 2 Coaches, 2 Parents, 3-4 Children, 3 Locations, 6 Sessions, 3 Requests, 2 Invoices, 1 MilestoneRule, 1 Resource, 2 Notifications.

---

## Implementation Notes
- Use `class-validator` + `class-transformer` on DTOs.  
- Store dates in UTC; display local (Asia/Colombo).  
- Use `pdf-lib` for light PDF exports.  
- Simple dev emails (log output).  
- Audit logs only major events.  
- Focus on correctness and maintainability over polish.

---

**Deliver the complete Admin MVP implementation as specified. If uncertain, prefer the simplest functional solution matching these requirements.**

