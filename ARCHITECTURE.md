# Soliton Campus Coder — Architecture Document

> **Version:** 1.0  
> **Date:** 2026-02-15  
> **Status:** Living Document

---

## 1. Overview

Soliton Campus Coder is an in-house coding assessment platform designed for campus recruitment. It replaces third-party platforms like HackerRank with a self-hosted solution that gives Soliton full control over the candidate evaluation pipeline.

### 1.1 Core Capabilities

| Capability | Description |
|---|---|
| **Question Bank** | Admins create coding questions with boilerplate code, public test cases (visible to candidates), and private test cases (hidden). |
| **Test Composition** | Tests are assembled from question bank items and include custom instructions. |
| **Candidate Invitations** | Admins bulk-invite candidates via email. Each invite creates a one-time personalised link tied to a specific date/time and duration. |
| **Proctored Test Session** | Tab-switch / browser-close / copy-paste detection. Violations auto-fail the test. |
| **Code Execution** | Candidate code is compiled and run server-side against test case inputs. Output is compared to expected output. |
| **Submission Analysis** | Per-candidate report: pass/fail per test case, submitted code, time taken, proctoring violations. |

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Framework | **Next.js 14+** (App Router) |
| Language | **TypeScript** (strict mode) |
| UI | **shadcn/ui** + **Tailwind CSS v3** |
| State Management | **TanStack Query** (server state) + **TanStack Store** (client state) |
| Database | **PostgreSQL 16** via **Prisma ORM** |
| Auth | **NextAuth.js v5** (Credentials provider for admins) |
| Email | **Nodemailer** (SMTP) / **Resend** |
| Code Execution | Local child process execution (V1) — see §8 for V1.1 security plan |
| Testing | **Vitest** (unit/integration) + **Playwright** (E2E) |
| Monorepo | Single Next.js app (no monorepo tooling needed for V1) |

---

## 3. High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Next.js App                        │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Admin Pages  │  │ Candidate    │  │  API Routes   │  │
│  │  /admin/*     │  │ Pages        │  │  /api/*       │  │
│  │              │  │ /test/*      │  │              │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                 │                 │          │
│  ┌──────┴─────────────────┴─────────────────┴───────┐  │
│  │                  Service Layer                    │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────────┐ │  │
│  │  │Question│ │  Test  │ │Invite  │ │  Execution │ │  │
│  │  │Service │ │Service │ │Service │ │  Engine    │ │  │
│  │  └────────┘ └────────┘ └────────┘ └────────────┘ │  │
│  └──────────────────────┬───────────────────────────┘  │
│                         │                              │
│  ┌──────────────────────┴───────────────────────────┐  │
│  │              Prisma ORM (Data Layer)              │  │
│  └──────────────────────┬───────────────────────────┘  │
└─────────────────────────┼───────────────────────────────┘
                          │
                ┌─────────┴─────────┐
                │   PostgreSQL DB   │
                └───────────────────┘
```

---

## 4. Data Models

### 4.1 Entity Relationship Diagram (Conceptual)

```
Admin ──┐
        │ creates
        ▼
   Question ──── TestCase (public / private)
        │
        │ selected into
        ▼
      Test ──── TestQuestion (join, with ordering)
        │
        │ invite
        ▼
  Invitation ──── Candidate
        │
        │ starts
        ▼
  TestSession ──── Submission (per question)
                      │
                      └── SubmissionResult (per test case)
```

### 4.2 Prisma Schema (Core Models)

```prisma
// ──── Auth & Users ────

model Admin {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  passwordHash  String
  createdAt     DateTime  @default(now())
  createdBy     String?   // admin who created this admin
}

model Candidate {
  id            String        @id @default(cuid())
  email         String        @unique
  name          String?
  createdAt     DateTime      @default(now())
  invitations   Invitation[]
  testSessions  TestSession[]
}

// ──── Question Bank ────

model Question {
  id              String          @id @default(cuid())
  title           String
  description     String          @db.Text    // Markdown
  difficulty      Difficulty      @default(MEDIUM)
  boilerplateCode String?         @db.Text
  language        Language        @default(C)
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt
  createdById     String
  testCases       TestCase[]
  testQuestions   TestQuestion[]
}

enum Difficulty {
  EASY
  MEDIUM
  HARD
}

enum Language {
  C
  CPP
}

model TestCase {
  id          String   @id @default(cuid())
  questionId  String
  question    Question @relation(fields: [questionId], references: [id], onDelete: Cascade)
  input       String   @db.Text
  output      String   @db.Text
  isPublic    Boolean  @default(false)
  order       Int      @default(0)
}

// ──── Tests ────

model Test {
  id            String          @id @default(cuid())
  title         String
  instructions  String          @db.Text    // Markdown shown to candidate
  durationMins  Int             // default duration in minutes
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
  createdById   String
  questions     TestQuestion[]
  invitations   Invitation[]
}

model TestQuestion {
  id          String   @id @default(cuid())
  testId      String
  test        Test     @relation(fields: [testId], references: [id], onDelete: Cascade)
  questionId  String
  question    Question @relation(fields: [questionId], references: [id], onDelete: Cascade)
  order       Int      @default(0)

  @@unique([testId, questionId])
}

// ──── Invitations ────

model Invitation {
  id              String          @id @default(cuid())
  token           String          @unique @default(cuid())  // one-time link token
  testId          String
  test            Test            @relation(fields: [testId], references: [id])
  candidateId     String
  candidate       Candidate       @relation(fields: [candidateId], references: [id])
  scheduledAt     DateTime        // date + time the test opens
  durationMins    Int             // duration for this specific invite
  status          InvitationStatus @default(PENDING)
  createdAt       DateTime        @default(now())
  emailSentAt     DateTime?
  testSession     TestSession?
}

enum InvitationStatus {
  PENDING       // email sent, not started
  STARTED       // test in progress
  COMPLETED     // candidate finished
  FAILED        // proctoring violation or time expired
  EXPIRED       // scheduledAt + durationMins passed without starting
}

// ──── Test Sessions ────

model TestSession {
  id              String           @id @default(cuid())
  invitationId    String           @unique
  invitation      Invitation       @relation(fields: [invitationId], references: [id])
  candidateId     String
  candidate       Candidate        @relation(fields: [candidateId], references: [id])
  startedAt       DateTime         @default(now())
  finishedAt      DateTime?
  status          SessionStatus    @default(IN_PROGRESS)
  failReason      String?          // e.g. "TAB_SWITCH", "BROWSER_CLOSE", "TIME_EXPIRED"
  submissions     Submission[]
  proctorEvents   ProctorEvent[]
}

enum SessionStatus {
  IN_PROGRESS
  COMPLETED
  FAILED
}

model ProctorEvent {
  id            String      @id @default(cuid())
  sessionId     String
  session       TestSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  eventType     String      // VISIBILITY_CHANGE, BLUR, COPY, PASTE, BEFOREUNLOAD
  timestamp     DateTime    @default(now())
  metadata      Json?
}

// ──── Submissions ────

model Submission {
  id              String              @id @default(cuid())
  sessionId       String
  session         TestSession         @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  questionId      String
  code            String              @db.Text
  language        Language
  submittedAt     DateTime            @default(now())
  results         SubmissionResult[]

  @@unique([sessionId, questionId])  // one submission per question per session
}

model SubmissionResult {
  id              String     @id @default(cuid())
  submissionId    String
  submission      Submission @relation(fields: [submissionId], references: [id], onDelete: Cascade)
  testCaseId      String
  passed          Boolean
  actualOutput    String?    @db.Text
  executionTimeMs Int?
  error           String?    @db.Text   // compilation or runtime error
}
```

---

## 5. Module Breakdown

### 5.1 Directory Structure

```
src/
├── app/
│   ├── (admin)/                    # Admin layout group
│   │   ├── admin/
│   │   │   ├── dashboard/          # Admin dashboard
│   │   │   ├── questions/          # Question CRUD
│   │   │   ├── tests/              # Test composition
│   │   │   ├── invitations/        # Invite management
│   │   │   ├── candidates/         # Candidate profiles & results
│   │   │   └── settings/           # Admin management (add admins)
│   │   └── layout.tsx              # Admin sidebar layout
│   ├── (candidate)/                # Candidate layout group
│   │   ├── test/
│   │   │   ├── [token]/            # Landing page (instructions + start)
│   │   │   └── session/[sessionId] # Active test page (code editor)
│   │   └── layout.tsx
│   ├── api/
│   │   ├── auth/                   # NextAuth routes
│   │   ├── questions/              # Question CRUD API
│   │   ├── tests/                  # Test CRUD API
│   │   ├── invitations/            # Invitation + bulk invite API
│   │   ├── sessions/               # Test session lifecycle API
│   │   ├── execute/                # Code execution API
│   │   └── submissions/            # Submission API
│   ├── layout.tsx
│   └── page.tsx                    # Landing / login redirect
├── components/
│   ├── ui/                         # shadcn/ui components
│   ├── admin/                      # Admin-specific components
│   ├── candidate/                  # Candidate-specific components
│   ├── code-editor/                # Monaco editor wrapper
│   └── common/                     # Shared components
├── lib/
│   ├── db.ts                       # Prisma client singleton
│   ├── auth.ts                     # NextAuth config
│   ├── email.ts                    # Email service
│   ├── execution/                  # Code execution engine
│   │   ├── index.ts                # Engine interface
│   │   ├── c-runner.ts             # C compiler + runner
│   │   ├── cpp-runner.ts           # C++ compiler + runner
│   │   └── types.ts                # Shared types
│   ├── validators/                 # Zod schemas
│   └── utils.ts                    # Utility functions
├── hooks/                          # Custom React hooks
│   ├── use-proctor.ts              # Tab switch / focus detection
│   ├── use-timer.ts                # Countdown timer
│   └── use-code-editor.ts          # Editor state management
├── stores/                         # TanStack Store definitions
│   └── test-session.ts             # Active test session state
├── types/                          # Global TypeScript types
└── __tests__/                      # Test files (mirrors src structure)
    ├── unit/
    ├── integration/
    └── e2e/
```

### 5.2 Core Modules

#### A. Code Execution Engine (`lib/execution/`)

The execution engine is designed with a **Strategy Pattern** to support multiple languages:

```typescript
// types.ts
interface ExecutionRequest {
  code: string;
  language: Language;
  input: string;
  timeoutMs?: number;   // default 10000
  memoryLimitMb?: number; // default 256
}

interface ExecutionResult {
  stdout: string;
  stderr: string;
  exitCode: number;
  executionTimeMs: number;
  timedOut: boolean;
  error?: string;
}

// index.ts — runner registry
interface LanguageRunner {
  compile(code: string, workDir: string): Promise<{ success: boolean; error?: string }>;
  run(executablePath: string, input: string, timeoutMs: number): Promise<ExecutionResult>;
}

const runners: Record<Language, LanguageRunner> = {
  C: new CRunner(),
  CPP: new CppRunner(),
  // Future: PYTHON: new PythonRunner(), JAVA: new JavaRunner(), etc.
};
```

**V1 Implementation:** Uses `child_process.execFile()` to invoke `gcc`/`g++` directly on the host. Each execution:
1. Writes source code to a temp file
2. Compiles with `gcc`/`g++`
3. Runs the binary, piping the test case input via stdin
4. Captures stdout, compares against expected output
5. Cleans up temp files

#### B. Proctoring Module (`hooks/use-proctor.ts`)

Client-side proctoring hooks that detect:

| Event | Browser API | Action |
|---|---|---|
| Tab switch | `document.visibilitychange` | Log event → Fail test |
| Window blur | `window.blur` | Log event → Fail test |
| Browser close/tab close | `window.beforeunload` | Log event → Fail test |
| Copy | `document.oncopy` | Prevent + log |
| Paste | `document.onpaste` | Prevent + log |
| Right-click | `contextmenu` event | Prevent |
| Keyboard shortcuts | `keydown` (Ctrl+C/V/Tab/Alt+Tab) | Prevent + log |

All events are logged to `ProctorEvent` table for the analysis report. The first violation triggers automatic test failure and navigation to a "Test Failed" page.

#### C. Email Service (`lib/email.ts`)

- Uses Nodemailer with configurable SMTP transport
- Supports bulk sending with rate limiting
- Email templates for: invitation, test reminder, test completion
- Each email contains the one-time personalised link: `{BASE_URL}/test/{invitation.token}`

---

## 6. API Routes

### 6.1 Admin APIs (Protected — require admin session)

| Method | Route | Description |
|---|---|---|
| POST | `/api/auth/register` | Create new admin (admin-only) |
| GET/POST | `/api/questions` | List / Create questions |
| GET/PUT/DELETE | `/api/questions/[id]` | Get / Update / Delete question |
| GET/POST | `/api/tests` | List / Create tests |
| GET/PUT/DELETE | `/api/tests/[id]` | Get / Update / Delete test |
| POST | `/api/invitations/bulk` | Bulk invite candidates |
| GET | `/api/invitations` | List invitations (filterable) |
| GET | `/api/candidates` | List candidates |
| GET | `/api/candidates/[id]` | Candidate profile + all test results |
| GET | `/api/candidates/[id]/sessions/[sessionId]` | Detailed session analysis |

### 6.2 Candidate APIs (Protected — require valid invitation/session)

| Method | Route | Description |
|---|---|---|
| GET | `/api/sessions/validate/[token]` | Validate invitation token |
| POST | `/api/sessions/start` | Start test session (invalidates invite link) |
| POST | `/api/execute` | Run code against public test cases ("Try") |
| POST | `/api/submissions` | Submit code for a question ("Submit") |
| POST | `/api/sessions/[id]/proctor-event` | Log proctoring event |
| POST | `/api/sessions/[id]/finish` | Finish test session |
| POST | `/api/sessions/[id]/fail` | Fail test session (proctoring violation) |

---

## 7. Key Flows

### 7.1 Admin Creates & Sends Invitations

```
Admin → Creates Questions (with test cases) 
      → Creates Test (selects questions, writes instructions)
      → Bulk Invite (uploads CSV or enters emails, picks test, sets date/time/duration)
      → System creates Candidate profiles (if needed)
      → System creates Invitation records
      → System sends emails with personalised links
```

### 7.2 Candidate Takes Test

```
Candidate clicks link → GET /test/[token]
  → Validate token (exists, not used, not expired)
  → Show landing page with instructions
  → Start button disabled until scheduledAt is reached
  
Candidate clicks Start → POST /api/sessions/start
  → Create TestSession record
  → Mark Invitation status = STARTED
  → Redirect to /test/session/[sessionId]
  → Start countdown timer
  → Activate proctoring hooks
  
During test:
  → Candidate writes code in Monaco editor
  → "Try" button → POST /api/execute (public test cases, results shown)
  → "Submit" button → Confirmation popup → POST /api/submissions (private test cases, results NOT shown)
  
Test ends:
  → All questions submitted OR timer expires
  → POST /api/sessions/[id]/finish
  → Generate analysis report
  → Show "Test Complete" page
  
Proctoring violation:
  → POST /api/sessions/[id]/proctor-event (log it)
  → POST /api/sessions/[id]/fail
  → Show "Test Failed" page
```

---

## 8. Security — V1 vs V1.1

### V1 (Current — Acceptable for Campus Recruitment)

Code is executed directly on the host using `child_process.execFile()`. This is acceptable because:
- Only campus recruitment candidates use the platform
- Risk of malicious actors is extremely low
- Code execution has basic safeguards: timeout limits, memory limits, temp directory isolation

**Basic V1 Safeguards:**
- Execution timeout (10 seconds default)
- Temp directory per execution, cleaned up after
- Non-root process execution
- `ulimit` constraints on child process (CPU time, file size, process count)

### V1.1 Security Hardening (Future)

The following changes should be made for V1.1:

1. **Docker Container Isolation**
   - Each code execution runs in a disposable Docker container
   - Container has no network access (`--network=none`)
   - Read-only filesystem except for `/tmp`
   - Resource limits enforced via cgroups (`--memory`, `--cpus`, `--pids-limit`)
   - Container is destroyed after execution

2. **Sandboxed Execution with nsjail / firejail**
   - Alternative to Docker — lighter weight
   - Uses Linux namespaces and seccomp-bpf to restrict syscalls
   - Blocks dangerous syscalls: `fork`, `exec` (beyond the compiled binary), network, filesystem access outside sandbox

3. **Code Analysis Pre-Check**
   - Static analysis of submitted code before compilation
   - Block `#include` of dangerous headers (e.g., `<sys/socket.h>`, `<unistd.h>` for `fork()`)
   - Block inline assembly
   - Reject code with known dangerous patterns

4. **Execution Queue**
   - Use a job queue (e.g., BullMQ + Redis) for code execution
   - Rate limit executions per candidate session
   - Prevent DoS from rapid submissions

5. **Network Isolation**
   - Execution environment has zero network access
   - Database and application server are on separate network segments

6. **Audit Logging**
   - Log all code executions with full input/output
   - Alert on suspicious patterns (e.g., attempted system commands)

---

## 9. Additional Considerations

### 9.1 Concurrency & Race Conditions
- Use database transactions for critical operations (starting sessions, submitting answers)
- Optimistic locking on submission records to prevent double-submit
- Server-side timer validation (don't trust client timer alone)

### 9.2 Time Synchronisation
- Server is the source of truth for all time calculations
- Client timer is for display only; server validates session duration on every API call
- Grace period of 30 seconds on timer expiry to account for network latency

### 9.3 Code Editor
- **Monaco Editor** (VS Code's editor) embedded in the test page
- Syntax highlighting for C/C++
- No auto-complete / IntelliSense (to test candidate's knowledge)
- Disable copy-paste within the editor during tests

### 9.4 Scalability Notes (V1)
- V1 targets ~50-100 concurrent candidates
- PostgreSQL handles this easily
- Code execution is the bottleneck; sequential execution per submission is fine for V1
- V1.1 should introduce a Redis-backed execution queue for higher concurrency

### 9.5 Deployment
- Single server deployment (VM or bare metal)
- `gcc` and `g++` must be installed on the server
- PostgreSQL can be on the same server or a managed instance
- Environment variables for all configuration (DB URL, SMTP, secrets)

---

## 10. Testing Strategy

| Layer | Tool | What We Test |
|---|---|---|
| **Unit** | Vitest | Service functions, validators, utilities, execution engine |
| **Integration** | Vitest + Prisma (test DB) | API routes, database operations, full execution pipeline |
| **E2E** | Playwright | Admin workflows, candidate test flow, proctoring behavior |
| **Component** | Vitest + React Testing Library | UI components in isolation |

### Key Test Scenarios
1. Complete admin flow: login → create question → create test → invite candidate
2. Complete candidate flow: open link → start test → write code → try → submit → finish
3. Proctoring: tab switch detection, browser close detection, copy-paste prevention
4. Edge cases: expired invitations, reused links, timer expiry, concurrent submissions
5. Code execution: successful compilation, compilation errors, runtime errors, timeout, correct/incorrect output

---

*This is a living document. Update it as the architecture evolves.*
