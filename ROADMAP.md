# Soliton Campus Coder — Development Roadmap

> **Version:** 1.0  
> **Date:** 2026-02-15  
> **Status:** Living Document

---

## Phase Overview

| Phase | Name | Description | Status |
|---|---|---|---|
| **0** | Project Setup | Next.js scaffold, DB, auth, UI foundation | ⬜ Not Started |
| **1** | Admin Core | Admin auth, admin management, question bank CRUD | ⬜ Not Started |
| **2** | Test Management | Test composition, invitation system, email delivery | ⬜ Not Started |
| **3** | Code Execution Engine | C/C++ compilation & execution, test case evaluation | ⬜ Not Started |
| **4** | Candidate Experience | Landing page, test session, code editor, proctoring | ⬜ Not Started |
| **5** | Submission & Analysis | Submit flow, result generation, candidate profiles | ⬜ Not Started |
| **6** | Polish & Testing | E2E tests, edge cases, UI polish, deployment prep | ⬜ Not Started |

---

## Phase 0: Project Setup

> Foundation — everything else builds on this.

### Tasks

- [ ] **0.1** Initialize Next.js 14+ project with TypeScript (App Router)
- [ ] **0.2** Configure Tailwind CSS v3
- [ ] **0.3** Install and configure shadcn/ui
- [ ] **0.4** Install TanStack Query + TanStack Store
- [ ] **0.5** Set up Prisma with PostgreSQL
- [ ] **0.6** Define complete Prisma schema (all models from architecture doc)
- [ ] **0.7** Run initial migration
- [ ] **0.8** Set up NextAuth.js v5 for admin authentication
- [ ] **0.9** Create Prisma client singleton (`lib/db.ts`)
- [ ] **0.10** Set up Vitest + React Testing Library
- [ ] **0.11** Set up Playwright for E2E
- [ ] **0.12** Create base layout components (admin sidebar, candidate layout)
- [ ] **0.13** Create seed script for development data
- [ ] **0.14** Configure environment variables (`.env.example`)

### Definition of Done
- App runs locally with `npm run dev`
- Database is created and migrated
- Admin can log in via NextAuth
- shadcn/ui components render correctly
- At least one test passes in Vitest

---

## Phase 1: Admin Core

> Admin can log in, manage other admins, and create questions.

### Tasks

- [ ] **1.1** Admin login page (`/admin/login`)
- [ ] **1.2** Admin dashboard page (`/admin/dashboard`) — stats overview
- [ ] **1.3** Admin management page (`/admin/settings`)
  - [ ] 1.3.1 List all admins
  - [ ] 1.3.2 Add new admin form (name, email, password)
  - [ ] 1.3.3 API: `POST /api/auth/register`
- [ ] **1.4** Question bank — List page (`/admin/questions`)
  - [ ] 1.4.1 Table with search, filter by difficulty/language
  - [ ] 1.4.2 API: `GET /api/questions`
- [ ] **1.5** Question bank — Create/Edit page (`/admin/questions/new`, `/admin/questions/[id]/edit`)
  - [ ] 1.5.1 Title, description (Markdown editor), difficulty, language selector
  - [ ] 1.5.2 Boilerplate code editor (Monaco)
  - [ ] 1.5.3 Public test cases section (add/remove input-output pairs)
  - [ ] 1.5.4 Private test cases section (add/remove input-output pairs)
  - [ ] 1.5.5 API: `POST /api/questions`, `PUT /api/questions/[id]`
- [ ] **1.6** Question bank — Delete question
  - [ ] 1.6.1 Confirmation dialog
  - [ ] 1.6.2 API: `DELETE /api/questions/[id]`
- [ ] **1.7** Question bank — View question detail (`/admin/questions/[id]`)
- [ ] **1.8** Unit tests for question CRUD APIs
- [ ] **1.9** Unit tests for admin registration API
- [ ] **1.10** Zod validation schemas for all question/admin inputs

### Definition of Done
- Admin can log in securely
- Admin can create, view, edit, delete questions
- Admin can add public and private test cases to questions
- Admin can add other admins
- All APIs have validation and error handling
- Unit tests pass

---

## Phase 2: Test Management & Invitations

> Admins compose tests and invite candidates.

### Tasks

- [ ] **2.1** Test list page (`/admin/tests`)
  - [ ] 2.1.1 Table with test name, question count, invitation count
  - [ ] 2.1.2 API: `GET /api/tests`
- [ ] **2.2** Test create/edit page (`/admin/tests/new`, `/admin/tests/[id]/edit`)
  - [ ] 2.2.1 Title, instructions (Markdown editor), default duration
  - [ ] 2.2.2 Question selector — search & add from question bank
  - [ ] 2.2.3 Question ordering (drag-and-drop)
  - [ ] 2.2.4 API: `POST /api/tests`, `PUT /api/tests/[id]`
- [ ] **2.3** Test detail page (`/admin/tests/[id]`)
  - [ ] 2.3.1 View test info, questions, invitations
- [ ] **2.4** Invitation management
  - [ ] 2.4.1 Invite form: select test, set date/time, set duration
  - [ ] 2.4.2 Single invite: enter email
  - [ ] 2.4.3 Bulk invite: CSV upload (email column) or textarea (one email per line)
  - [ ] 2.4.4 API: `POST /api/invitations/bulk`
- [ ] **2.5** Candidate profile auto-creation on invite
- [ ] **2.6** Email service (`lib/email.ts`)
  - [ ] 2.6.1 SMTP / Resend configuration
  - [ ] 2.6.2 Invitation email template with personalised link
  - [ ] 2.6.3 Rate limiting for bulk sends
- [ ] **2.7** Invitation list page (`/admin/invitations`)
  - [ ] 2.7.1 Filter by test, status, date
  - [ ] 2.7.2 Show status badges (Pending, Started, Completed, Failed, Expired)
- [ ] **2.8** Candidate list page (`/admin/candidates`)
  - [ ] 2.8.1 List all candidates with test history summary
- [ ] **2.9** Unit tests for test CRUD APIs
- [ ] **2.10** Unit tests for invitation/bulk invite APIs
- [ ] **2.11** Unit tests for email service

### Definition of Done
- Admin can create tests with questions and instructions
- Admin can bulk invite candidates
- Emails are sent with personalised one-time links
- Candidate profiles are auto-created
- All APIs validated and tested

---

## Phase 3: Code Execution Engine

> Server-side C/C++ compilation and execution.

### Tasks

- [ ] **3.1** Define execution engine interface (`lib/execution/types.ts`)
- [ ] **3.2** Implement C runner (`lib/execution/c-runner.ts`)
  - [ ] 3.2.1 Write source to temp file
  - [ ] 3.2.2 Compile with `gcc`
  - [ ] 3.2.3 Execute binary with stdin input
  - [ ] 3.2.4 Capture stdout/stderr
  - [ ] 3.2.5 Enforce timeout and cleanup
- [ ] **3.3** Implement C++ runner (`lib/execution/cpp-runner.ts`)
  - [ ] 3.3.1 Same as C runner but with `g++`
- [ ] **3.4** Runner registry / factory (`lib/execution/index.ts`)
- [ ] **3.5** Execution API route (`POST /api/execute`)
  - [ ] 3.5.1 Accept code + language + test case inputs
  - [ ] 3.5.2 Compile, run, compare output
  - [ ] 3.5.3 Return pass/fail per test case with details
- [ ] **3.6** "Try" flow — run against public test cases, return results to candidate
- [ ] **3.7** "Submit" flow — run against private test cases, store results but don't expose to candidate
- [ ] **3.8** Basic safety: timeouts, ulimits, temp directory isolation
- [ ] **3.9** Unit tests for C runner
- [ ] **3.10** Unit tests for C++ runner
- [ ] **3.11** Integration tests for execution API

### Definition of Done
- C and C++ code can be compiled and executed
- Test case input is piped to stdin, output is compared
- Timeouts prevent infinite loops
- Compilation errors are captured and returned
- Adding a new language requires only creating a new runner class

---

## Phase 4: Candidate Experience

> The entire candidate-facing flow.

### Tasks

- [ ] **4.1** Test landing page (`/test/[token]`)
  - [ ] 4.1.1 Validate invitation token (exists, not used, not expired)
  - [ ] 4.1.2 Display test instructions (rendered Markdown)
  - [ ] 4.1.3 Display scheduled date/time and duration
  - [ ] 4.1.4 "Start" button — disabled until `scheduledAt` is reached
  - [ ] 4.1.5 Countdown to start time (if not yet reached)
  - [ ] 4.1.6 Show error states: invalid token, already used, expired
- [ ] **4.2** Start test flow
  - [ ] 4.2.1 API: `POST /api/sessions/start`
  - [ ] 4.2.2 Create TestSession, mark invitation as STARTED
  - [ ] 4.2.3 Redirect to test page
- [ ] **4.3** Test page (`/test/session/[sessionId]`)
  - [ ] 4.3.1 Question navigation sidebar (numbered, with status indicators)
  - [ ] 4.3.2 Monaco code editor with C/C++ syntax highlighting
  - [ ] 4.3.3 Pre-populated boilerplate code
  - [ ] 4.3.4 "Try" button — run against public test cases, show results
  - [ ] 4.3.5 "Submit" button — confirmation popup → submit against private test cases
  - [ ] 4.3.6 Test case results panel (for "Try" — shows input, expected output, actual output)
  - [ ] 4.3.7 Countdown timer (top of page)
  - [ ] 4.3.8 Question status: Not Attempted / Attempted / Submitted
- [ ] **4.4** Proctoring implementation (`hooks/use-proctor.ts`)
  - [ ] 4.4.1 `visibilitychange` listener — detect tab switch
  - [ ] 4.4.2 `blur` listener — detect window focus loss
  - [ ] 4.4.3 `beforeunload` listener — detect browser/tab close
  - [ ] 4.4.4 Copy/paste prevention (`oncopy`, `onpaste`, `oncut`)
  - [ ] 4.4.5 Right-click prevention (`contextmenu`)
  - [ ] 4.4.6 Keyboard shortcut interception (Ctrl+C, Ctrl+V, Alt+Tab, etc.)
  - [ ] 4.4.7 Log all events to API: `POST /api/sessions/[id]/proctor-event`
  - [ ] 4.4.8 Auto-fail on first violation: `POST /api/sessions/[id]/fail`
  - [ ] 4.4.9 Show "Test Failed" page
- [ ] **4.5** Timer implementation (`hooks/use-timer.ts`)
  - [ ] 4.5.1 Countdown from duration
  - [ ] 4.5.2 Server-side timer validation on every API call
  - [ ] 4.5.3 Auto-submit all questions when timer expires
  - [ ] 4.5.4 Warning at 5 minutes remaining
- [ ] **4.6** Test completion page
  - [ ] 4.6.1 "Thank you" message
  - [ ] 4.6.2 No results shown to candidate
- [ ] **4.7** Test failed page
  - [ ] 4.7.1 "Test failed due to policy violation" message
- [ ] **4.8** TanStack Store for test session state
- [ ] **4.9** Component tests for code editor, timer, question navigator

### Definition of Done
- Candidate can open invite link, see instructions, start test when time arrives
- Candidate can write code, try against public test cases, submit
- Used invite link is invalidated
- Proctoring detects violations and fails the test
- Timer auto-submits when expired
- Submit has confirmation popup

---

## Phase 5: Submission & Analysis

> Results analysis and admin viewing.

### Tasks

- [ ] **5.1** Submission processing
  - [ ] 5.1.1 Run submitted code against all private test cases
  - [ ] 5.1.2 Store results (passed/failed per test case, actual output, execution time)
  - [ ] 5.1.3 Handle compilation errors, runtime errors, timeouts
- [ ] **5.2** Session completion handler
  - [ ] 5.2.1 Calculate overall pass/fail
  - [ ] 5.2.2 Calculate total time taken
  - [ ] 5.2.3 Aggregate proctoring events
- [ ] **5.3** Candidate profile page (`/admin/candidates/[id]`)
  - [ ] 5.3.1 Candidate info (name, email)
  - [ ] 5.3.2 List of all test sessions with status
  - [ ] 5.3.3 Summary stats (tests taken, pass rate)
- [ ] **5.4** Session analysis page (`/admin/candidates/[id]/sessions/[sessionId]`)
  - [ ] 5.4.1 Overview: test name, status (completed/failed), total time
  - [ ] 5.4.2 Per-question breakdown:
    - Submitted code (syntax highlighted)
    - Public test case results (pass/fail, input/expected/actual output)
    - Private test case results (pass/fail, input/expected/actual output)
    - Time of submission
  - [ ] 5.4.3 Proctoring log (timeline of events)
  - [ ] 5.4.4 If failed: failure reason (tab switch, browser close, time expired, etc.)
- [ ] **5.5** Dashboard updates
  - [ ] 5.5.1 Total tests, total candidates, recent activity
  - [ ] 5.5.2 Quick access to recent sessions
- [ ] **5.6** Unit tests for submission processing
- [ ] **5.7** Unit tests for analysis generation

### Definition of Done
- Admin can view detailed analysis of any candidate's test session
- Analysis includes code, test case results, time, and proctoring events
- Dashboard shows useful overview stats

---

## Phase 6: Polish & Testing

> Make it production-ready.

### Tasks

- [ ] **6.1** E2E test: Complete admin flow
  - [ ] 6.1.1 Login → Create question → Create test → Invite candidate
- [ ] **6.2** E2E test: Complete candidate flow
  - [ ] 6.2.1 Open link → Start → Write code → Try → Submit → Finish
- [ ] **6.3** E2E test: Proctoring violation
  - [ ] 6.3.1 Start test → Switch tab → Verify test fails
- [ ] **6.4** E2E test: Timer expiry
  - [ ] 6.4.1 Start test → Wait for timer → Verify auto-submit
- [ ] **6.5** Edge case handling
  - [ ] 6.5.1 Expired invitation token
  - [ ] 6.5.2 Reused invitation token
  - [ ] 6.5.3 Network failure during test
  - [ ] 6.5.4 Rapid double-click on submit
  - [ ] 6.5.5 Very large code submission
  - [ ] 6.5.6 Code with infinite loops (timeout)
- [ ] **6.6** UI polish
  - [ ] 6.6.1 Loading states and skeletons
  - [ ] 6.6.2 Error toasts and notifications
  - [ ] 6.6.3 Responsive design for admin panel
  - [ ] 6.6.4 Accessibility improvements
  - [ ] 6.6.5 Dark mode support
- [ ] **6.7** Performance
  - [ ] 6.7.1 API response time optimization
  - [ ] 6.7.2 Database query optimization (indexes)
  - [ ] 6.7.3 Code editor loading optimization
- [ ] **6.8** Deployment documentation
  - [ ] 6.8.1 Server requirements
  - [ ] 6.8.2 Environment variable documentation
  - [ ] 6.8.3 Database setup guide
  - [ ] 6.8.4 Deployment steps

### Definition of Done
- All E2E tests pass
- Edge cases handled gracefully
- UI is polished and responsive
- Documentation is complete
- App is deployable

---

## Current Progress

| Task | Status | Notes |
|---|---|---|
| Phase 0 — Project Setup | ⬜ Not Started | — |

---

## Notes & Decisions

- **Monaco Editor** chosen over CodeMirror for better C/C++ support and closer VS Code experience
- **Server-side timer validation** is critical — never trust client-side timer alone
- **One submission per question** — once submitted, code cannot be changed for that question
- **Proctoring is aggressive** — first violation = auto-fail (as per requirements)
- **V1 targets ~50-100 concurrent candidates** — no need for execution queue yet

---

*This document will be updated as development progresses. Check off tasks as they are completed.*
