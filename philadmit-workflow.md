# PhilAdmit — Platform Workflow Documentation
> Central Admissions Platform for Philippine Higher Education Institutions
> Modeled after UCAS (UK) · Governed under CHED · Built for Filipino Students

---

## Table of Contents

1. [Platform Actors](#1-platform-actors)
2. [Applicant Workflow](#2-applicant-workflow)
3. [HEI Admin Workflow](#3-hei-admin-workflow)
4. [CHED Regulator Workflow](#4-ched-regulator-workflow)
5. [Platform Superadmin Workflow](#5-platform-superadmin-workflow)
6. [Core System Workflows](#6-core-system-workflows)
   - [Application Engine](#61-application-engine)
   - [Offer Management](#62-offer-management)
   - [Document Management](#63-document-management)
   - [Notification System](#64-notification-system)
   - [Deadline Engine](#65-deadline-engine)
   - [Audit Log](#66-audit-log)
7. [Data Flow Diagram](#7-data-flow-diagram)
8. [Status State Machine](#8-status-state-machine)
9. [Tech Stack & Environment](#9-tech-stack--environment)

---

## 1. Platform Actors

| Actor | Role | Access Level |
|-------|------|-------------|
| **Applicant** | Filipino student applying to HEIs | Own applications only |
| **HEI Admin** | University/college admissions officer | Own institution's applicants |
| **CHED Regulator** | National oversight body | Read-only, all institutions |
| **Platform Superadmin** | PhilAdmit internal operations | Full system access |

---

## 2. Applicant Workflow

### 2.1 Registration & Profile Setup

```
START
  │
  ▼
Register with email + password
  │
  ▼
Verify email address
  │
  ▼
Complete Profile
  ├── Full legal name (as per PSA)
  ├── Date of birth
  ├── Sex assigned at birth
  ├── Nationality
  ├── Home address (PSGC region, province, city/municipality)
  ├── Contact number (PH mobile)
  └── LRN (Learner Reference Number — 12-digit, unique, required)
  │
  ▼
Profile saved → Proceed to Application
```

### 2.2 Multi-Step Application Form

**Step 1 — Personal Information**
- Full name, birthdate, sex, nationality
- PSGC-based region and address
- Emergency contact details

**Step 2 — Academic Background**
- Senior High School (SHS) name and school ID
- SHS Strand: `STEM | ABM | HUMSS | GAS | TVL | Sports | Arts and Design`
- General Weighted Average (GWA) from Grade 11 and 12
- SHS graduation year

**Step 3 — Program Choices (up to 5, ranked)**

```
Choice 1 (First Priority)  →  [ Select HEI ] → [ Select Program ]
Choice 2 (Second Priority) →  [ Select HEI ] → [ Select Program ]
Choice 3 (Third Priority)  →  [ Select HEI ] → [ Select Program ]
Choice 4 (Fourth Priority) →  [ Select HEI ] → [ Select Program ]
Choice 5 (Fifth Priority)  →  [ Select HEI ] → [ Select Program ]

Rules enforced:
  ✗ No duplicate HEI in the same application
  ✗ Maximum of 5 choices per application cycle
  ✓ Ranked order is applicant-controlled
```

**Step 4 — Document Uploads**

| Document | Type | Required |
|----------|------|----------|
| SHS Report Card (Form 138) | Academic | ✅ Yes |
| PSA Birth Certificate | Government-issued | ✅ Yes |
| Good Moral Certificate | Character | ✅ Yes |
| SHS Diploma or Completion Certificate | Academic | ✅ Yes |
| Passport-size Photo | Identity | ✅ Yes |

> ⚠️ PSA-issued documents are flagged as government-issued and subject to stricter access controls.

**Step 5 — Review & Submit**
- Preview all entries across Steps 1–4
- Confirm accuracy declaration (checkboxes)
- Click **Submit Application**
- Application ID generated
- Confirmation email + SMS sent

### 2.3 Post-Submission Applicant Actions

```
Application Submitted
  │
  ├── [Monitor status per HEI choice]
  │     ├── Pending → Conditional Offer → Firm Offer
  │     └── Pending → Rejected
  │
  ├── [Receive Offer]
  │     ├── Accept Firm Offer
  │     │     └── All other choices auto-set to: Withdrawn
  │     └── Decline Offer
  │           └── Next ranked choice remains active
  │
  └── [Withdraw Application]
        └── All choices cancelled (irreversible)
```

---

## 3. HEI Admin Workflow

### 3.1 Institution Onboarding
> Performed by Platform Superadmin. HEI Admin receives credentials after onboarding.

### 3.2 Applicant Review Dashboard

```
Login → HEI Dashboard
  │
  ▼
View Applicant Queue
  ├── Filter by: Program | Status | Region | SHS Strand | GWA range
  ├── Sort by: Date submitted | GWA | Name
  └── Search by: Applicant name | LRN | Application ID
  │
  ▼
Open Applicant Profile
  ├── View personal information
  ├── View academic records (GWA, strand)
  ├── Download/view uploaded documents
  │     └── Access logged in Audit Log with timestamp + HEI Admin ID
  └── View ranked choice position (e.g., "This applicant listed your institution as Choice 2")
  │
  ▼
Update Offer Status
  ├── Conditional Offer  → Applicant notified (email + SMS)
  ├── Firm Offer         → Applicant notified (email + SMS)
  └── Rejected           → Applicant notified (email + SMS)
  │
  ▼
All status changes timestamped → Written to Audit Log
```

### 3.3 Deadline Configuration
- Set application open date and close date per program
- After close date: no new applications accepted for that program
- System enforces cutoff automatically

### 3.4 Reporting
- Export applicant list (CSV) filtered by status or program
- View acceptance rate per program
- View regional distribution of applicants

---

## 4. CHED Regulator Workflow

> Read-only access. No ability to modify application data or offer statuses.

```
Login → CHED Analytics Dashboard
  │
  ├── Summary Cards
  │     ├── Total applications nationwide (current cycle)
  │     ├── Total HEIs onboarded on PhilAdmit
  │     ├── Total offers issued (Conditional + Firm)
  │     └── Total accepted placements
  │
  ├── Regional Breakdown Chart
  │     └── Applications by PSGC region (bar/choropleth)
  │
  ├── HEI Performance Table
  │     ├── HEI name
  │     ├── Total applicants
  │     ├── Acceptance rate (%)
  │     └── Programs with most demand
  │
  └── Export Reports (PDF / CSV)
        └── For official CHED documentation
```

---

## 5. Platform Superadmin Workflow

```
Login → Superadmin Panel
  │
  ├── HEI Management
  │     ├── Onboard new HEI (name, CHED accreditation number, region, programs offered)
  │     ├── Suspend or deactivate HEI account
  │     └── Assign HEI Admin accounts per institution
  │
  ├── User Account Management
  │     ├── View all users by role
  │     ├── Reset passwords
  │     ├── Deactivate compromised accounts
  │     └── Assign/revoke roles
  │
  ├── System Configuration
  │     ├── Set global application cycle dates
  │     ├── Configure max choices per application (default: 5)
  │     └── Toggle maintenance mode
  │
  └── Audit Log Viewer
        ├── Search by actor, action type, date range
        └── Export for compliance reporting
```

---

## 6. Core System Workflows

### 6.1 Application Engine

```
Applicant submits application
  │
  ▼
System validates:
  ├── LRN is unique and 12 digits ✓
  ├── Max 5 HEI choices ✓
  ├── No duplicate HEIs in choices ✓
  ├── All required documents uploaded ✓
  └── Application window is open for each chosen HEI ✓
  │
  ▼
Application record created in DB
  │
  ├── One Application → up to 5 HEIChoice records
  └── Each HEIChoice → OfferStatus initialized to: PENDING
  │
  ▼
Application ID generated (format: PHA-YYYY-XXXXXXX)
  │
  ▼
Confirmation notification triggered (email + SMS)
```

### 6.2 Offer Management

```
HEI Admin issues Firm Offer to applicant
  │
  ▼
Applicant receives notification
  │
  ▼
Applicant decision:

  ┌─────────────────────┐      ┌──────────────────────────┐
  │   ACCEPT OFFER      │      │   DECLINE OFFER          │
  └────────┬────────────┘      └──────────┬───────────────┘
           │                              │
           ▼                              ▼
  This HEIChoice → ACCEPTED     This HEIChoice → DECLINED
           │                              │
           ▼                              ▼
  All other HEIChoices          Remaining choices stay
  in same application →         ACTIVE (next ranked
  auto-set to WITHDRAWN         choice takes priority)
           │
           ▼
  Applicant cannot accept
  another offer this cycle
  (enforced at DB level)
```

### 6.3 Document Management

```
Applicant uploads document
  │
  ▼
File validation:
  ├── Accepted formats: PDF, JPG, PNG
  ├── Max file size: 5MB per file
  └── Required documents checklist enforced before submission
  │
  ▼
File stored in Supabase Storage / AWS S3
  (toggled via ENV: STORAGE_PROVIDER)
  │
  ▼
Secure signed URL generated per access request
  │
  ▼
HEI Admin accesses document
  └── Access event written to Audit Log:
        { actor: HEI_ADMIN_ID, action: DOCUMENT_VIEW,
          document_id: X, applicant_id: Y, timestamp: Z }

PSA-issued documents (Birth Certificate):
  └── Extra access control: only HEI Admin of chosen HEI can view
```

### 6.4 Notification System

| Trigger Event | Email | SMS (Semaphore PH) |
|---------------|-------|-------------------|
| Application submitted | ✅ | ✅ |
| Conditional Offer issued | ✅ | ✅ |
| Firm Offer issued | ✅ | ✅ |
| Application rejected | ✅ | ✅ |
| Offer accepted (confirmation) | ✅ | ✅ |
| Document missing reminder | ✅ | ✅ |
| Deadline approaching (3 days) | ✅ | ✅ |
| Application withdrawn | ✅ | — |

> SMS integration via **Semaphore PH** — stubbed in scaffold, marked `// TODO: configure Semaphore API key`

### 6.5 Deadline Engine

```
Each HEI Program has:
  ├── application_open_date
  └── application_close_date

System checks on every application submission:
  └── current_date >= open AND current_date <= close → ALLOW
      current_date > close → REJECT with: "Application window closed"

Automated jobs (cron):
  ├── T-3 days: send deadline reminder notification to pending applicants
  └── T+0 (close date): auto-close program, reject unsubmitted drafts
```

### 6.6 Audit Log

Every entry contains:

```
{
  id:           UUID,
  actor_id:     User ID,
  actor_role:   APPLICANT | HEI_ADMIN | CHED | SUPERADMIN,
  action:       ENUM (APPLICATION_SUBMITTED | STATUS_UPDATED |
                       DOCUMENT_VIEWED | OFFER_ACCEPTED |
                       OFFER_DECLINED | APPLICATION_WITHDRAWN |
                       HEI_ONBOARDED | USER_DEACTIVATED),
  target_type:  Application | HEIChoice | Document | User | HEI,
  target_id:    UUID,
  metadata:     JSON (old_value, new_value, IP address),
  created_at:   TIMESTAMP (UTC+8, PH timezone)
}
```

---

## 7. Data Flow Diagram

```
┌─────────────┐     submits      ┌──────────────────┐
│  Applicant  │ ───────────────► │   Application    │
└─────────────┘                  │   (1 per cycle)  │
                                 └────────┬─────────┘
                                          │ has up to 5
                                          ▼
                                 ┌──────────────────┐
                                 │   HEI Choices    │
                                 │  (ranked 1–5)    │
                                 └────────┬─────────┘
                                          │ each has
                                          ▼
                                 ┌──────────────────┐
                                 │   Offer Status   │
                                 │ PENDING →        │
                                 │ CONDITIONAL →    │
                                 │ FIRM →           │
                                 │ ACCEPTED /       │
                                 │ REJECTED /       │
                                 │ WITHDRAWN        │
                                 └────────┬─────────┘
                                          │ updated by
                                          ▼
                                 ┌──────────────────┐
                                 │   HEI Admin      │
                                 │   (per HEI)      │
                                 └──────────────────┘

Documents:   Applicant → uploads → S3/Supabase → HEI Admin views
Notifications: System → triggers → Email (Resend) + SMS (Semaphore PH)
Audit:       Every action → Audit Log → Superadmin / CHED can view
```

---

## 8. Status State Machine

```
                    ┌──────────┐
                    │ PENDING  │  ◄── Initial state on submission
                    └────┬─────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
   ┌──────────────┐  ┌────────┐  ┌──────────┐
   │ CONDITIONAL  │  │ FIRM   │  │ REJECTED │  ◄── Terminal
   │    OFFER     │  │ OFFER  │  └──────────┘
   └──────┬───────┘  └───┬────┘
          │              │
          ▼              ├──────────────┐
   ┌──────────────┐      ▼              ▼
   │ FIRM  OFFER  │  ┌──────────┐  ┌──────────┐
   │  (upgraded)  │  │ ACCEPTED │  │ DECLINED │
   └──────────────┘  │(terminal)│  └──────────┘
                     └──────────┘
                          │
                          ▼
              All other choices → WITHDRAWN (auto)


   WITHDRAWN ◄── Set automatically when another offer is ACCEPTED
             ◄── OR when Applicant manually withdraws entire application
```

---

## 9. Tech Stack & Environment

### Stack Summary

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 14 (App Router), TypeScript, Tailwind CSS, shadcn/ui |
| Backend | Next.js API Routes, TypeScript |
| Database | PostgreSQL + Prisma ORM |
| Auth | NextAuth.js with role-based session claims |
| File Storage | Supabase Storage or AWS S3 (ENV toggle) |
| Email | Resend (abstracted behind provider interface) |
| SMS | Semaphore PH (stubbed, TODO) |
| Hosting | Vercel (frontend) + Railway or Supabase (DB) |

### Route Groups by Actor

| Route Group | Actor | Example Routes |
|-------------|-------|---------------|
| `(applicant)` | Applicant | `/apply`, `/dashboard`, `/status` |
| `(hei)` | HEI Admin | `/hei/applicants`, `/hei/offers`, `/hei/reports` |
| `(ched)` | CHED Regulator | `/ched/analytics`, `/ched/hei-list` |
| `(admin)` | Superadmin | `/admin/heis`, `/admin/users`, `/admin/logs` |

### Required Environment Variables

```env
# Database
DATABASE_URL=

# Auth
NEXTAUTH_SECRET=
NEXTAUTH_URL=

# File Storage (set to "supabase" or "s3")
STORAGE_PROVIDER=supabase

# Supabase (if STORAGE_PROVIDER=supabase)
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# AWS S3 (if STORAGE_PROVIDER=s3)
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_S3_BUCKET=
AWS_REGION=

# Email
RESEND_API_KEY=

# SMS — Semaphore PH (TODO: configure)
SEMAPHORE_API_KEY=
SEMAPHORE_SENDER_NAME=PhilAdmit

# App
NEXT_PUBLIC_APP_URL=
APP_ENV=development
```

---

> **PhilAdmit** is free to all applicants. No payment gateway is integrated.
> Built to serve every Filipino student's path to higher education — from Batanes to BARMM.
