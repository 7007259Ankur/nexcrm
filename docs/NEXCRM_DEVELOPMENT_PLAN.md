# NexCRM — Comprehensive Development & Implementation Plan

**Document Version:** 1.0.0  
**Status:** Approved Roadmap  
**Target Milestone 1:** Organization Registration -> User Creation -> Login -> Tenant Isolation Verification

---

## 1. Project Directory Architecture

```
D:\NexCRM\
├── backend/
│   ├── alembic/
│   │   ├── env.py
│   │   ├── script.py.mako
│   │   └── versions/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                     # App factory, CORS, exception handlers
│   │   ├── config.py                   # Pydantic Settings
│   │   ├── database.py                 # Async engine & session factory
│   │   ├── dependencies.py             # get_db, get_current_user, get_current_org
│   │   ├── exceptions.py               # Custom domain exceptions
│   │   │
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── deps.py                 # Permission dependencies
│   │   │   └── v1/
│   │   │       ├── __init__.py
│   │   │       ├── router.py           # Master v1 router aggregating all endpoints
│   │   │       ├── auth.py
│   │   │       ├── organizations.py
│   │   │       ├── users.py
│   │   │       ├── roles.py
│   │   │       ├── leads.py
│   │   │       ├── contacts.py
│   │   │       ├── companies.py
│   │   │       ├── deals.py
│   │   │       ├── pipelines.py
│   │   │       ├── activities.py
│   │   │       ├── tasks.py
│   │   │       ├── notes.py
│   │   │       ├── tags.py
│   │   │       ├── custom_fields.py
│   │   │       ├── workflows.py
│   │   │       ├── reports.py
│   │   │       ├── notifications.py
│   │   │       └── attachments.py
│   │   │
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   ├── security.py             # Argon2 password hashing & JWT handling
│   │   │   ├── permissions.py          # Permission definitions & enforcement
│   │   │   ├── pagination.py           # Standard pagination math & schemas
│   │   │   └── storage.py              # File storage interface & local driver
│   │   │
│   │   ├── middleware/
│   │   │   ├── __init__.py
│   │   │   └── tenant.py               # Tenant ContextVar & PostgreSQL RLS setup
│   │   │
│   │   ├── models/
│   │   │   ├── __init__.py             # Exports all models for Alembic
│   │   │   ├── base.py                 # DeclarativeBase, UUID & Audit mixins
│   │   │   ├── organization.py         # Organization & Invitation
│   │   │   ├── user.py                 # User & RefreshToken
│   │   │   ├── rbac.py                 # Role, Permission, RolePermission, UserRole
│   │   │   ├── lead.py                 # Lead model
│   │   │   ├── contact.py              # Contact model
│   │   │   ├── company.py              # Company model
│   │   │   ├── deal.py                 # Pipeline, DealStage, Deal
│   │   │   ├── activity.py             # Activity & Task models
│   │   │   ├── note.py                 # Note model
│   │   │   ├── meta.py                 # Tag, EntityTag, CustomField, CustomFieldValue
│   │   │   ├── workflow.py             # WorkflowRule, WorkflowExecution
│   │   │   ├── notification.py         # Notification model
│   │   │   ├── attachment.py           # Attachment model
│   │   │   └── audit_log.py            # AuditLog model
│   │   │
│   │   ├── repositories/
│   │   │   ├── __init__.py
│   │   │   ├── base.py                 # Generic CRUD repo with auto tenant filtering
│   │   │   ├── organization_repo.py
│   │   │   ├── user_repo.py
│   │   │   ├── role_repo.py
│   │   │   ├── lead_repo.py
│   │   │   ├── contact_repo.py
│   │   │   ├── company_repo.py
│   │   │   ├── deal_repo.py
│   │   │   ├── pipeline_repo.py
│   │   │   ├── activity_repo.py
│   │   │   ├── task_repo.py
│   │   │   └── audit_repo.py
│   │   │
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── common.py               # Envelope, PaginatedResponse, ErrorResponse
│   │   │   ├── auth.py
│   │   │   ├── organization.py
│   │   │   ├── user.py
│   │   │   ├── rbac.py
│   │   │   ├── lead.py
│   │   │   ├── contact.py
│   │   │   ├── company.py
│   │   │   ├── deal.py
│   │   │   ├── pipeline.py
│   │   │   ├── activity.py
│   │   │   ├── task.py
│   │   │   ├── workflow.py
│   │   │   └── report.py
│   │   │
│   │   └── services/
│   │       ├── __init__.py
│   │       ├── auth_service.py
│   │       ├── organization_service.py
│   │       ├── user_service.py
│   │       ├── lead_service.py         # Includes atomic lead conversion
│   │       ├── contact_service.py
│   │       ├── company_service.py
│   │       ├── deal_service.py
│   │       ├── pipeline_service.py
│   │       ├── activity_service.py
│   │       ├── task_service.py
│   │       ├── workflow_service.py
│   │       ├── report_service.py
│   │       └── notification_service.py
│   │
│   ├── tests/
│   │   ├── conftest.py
│   │   ├── test_auth.py
│   │   ├── test_tenancy_isolation.py
│   │   ├── test_leads.py
│   │   ├── test_lead_conversion.py
│   │   └── test_deals.py
│   ├── pyproject.toml
│   ├── requirements.txt
│   ├── alembic.ini
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── (auth)/
│   │   │   │   ├── login/page.tsx
│   │   │   │   ├── register/page.tsx
│   │   │   │   └── layout.tsx
│   │   │   ├── (dashboard)/
│   │   │   │   ├── dashboard/page.tsx
│   │   │   │   ├── leads/page.tsx
│   │   │   │   ├── leads/[id]/page.tsx
│   │   │   │   ├── contacts/page.tsx
│   │   │   │   ├── companies/page.tsx
│   │   │   │   ├── deals/page.tsx
│   │   │   │   ├── pipelines/page.tsx
│   │   │   │   ├── activities/page.tsx
│   │   │   │   ├── tasks/page.tsx
│   │   │   │   ├── reports/page.tsx
│   │   │   │   ├── settings/page.tsx
│   │   │   │   └── layout.tsx
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── components/
│   │   │   ├── ui/                    # Button, Input, Modal, Badge, Dropdown, Table
│   │   │   ├── layout/
│   │   │   │   ├── sidebar.tsx
│   │   │   │   ├── header.tsx
│   │   │   │   └── breadcrumbs.tsx
│   │   │   ├── data-table/
│   │   │   │   ├── data-table.tsx
│   │   │   │   ├── pagination.tsx
│   │   │   │   └── column-header.tsx
│   │   │   ├── leads/
│   │   │   │   ├── lead-form.tsx
│   │   │   │   ├── lead-convert-modal.tsx
│   │   │   │   └── lead-status-badge.tsx
│   │   │   ├── deals/
│   │   │   │   ├── kanban-board.tsx
│   │   │   │   ├── kanban-column.tsx
│   │   │   │   └── deal-card.tsx
│   │   │   └── charts/
│   │   ├── lib/
│   │   │   ├── api-client.ts          # Axios client with interceptors
│   │   │   ├── auth.ts                # Token cookie/localStorage management
│   │   │   └── utils.ts
│   │   ├── hooks/
│   │   │   ├── use-auth.ts
│   │   │   └── use-permissions.ts
│   │   ├── store/
│   │   │   └── auth-store.ts          # Zustand session state
│   │   └── types/
│   │       ├── api.ts
│   │       ├── crm.ts
│   │       └── auth.ts
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.ts
│   └── next.config.ts
│
└── docs/
    ├── NEXCRM_SYSTEM_DESIGN.md
    ├── NEXCRM_DATABASE_DESIGN.md
    ├── NEXCRM_API_SPECIFICATION.md
    ├── NEXCRM_BUSINESS_LOGIC.md
    └── NEXCRM_DEVELOPMENT_PLAN.md
```

---

## 2. Phase-by-Phase Implementation Roadmap

### Phase 1: Foundation (Weeks 1–2)

#### Sprint 1.1: Backend Project Setup & Database Connection
- [ ] **Task 1.1.1:** Initialize Python environment with `pyproject.toml` and `requirements.txt` (FastAPI, SQLAlchemy 2.0, AsyncPG, Alembic, Pydantic v2, Passlib, PyJWT).
- [ ] **Task 1.1.2:** Configure `app/config.py` using `pydantic-settings.BaseSettings` for database URL, JWT secrets, and environment flags.
- [ ] **Task 1.1.3:** Setup asynchronous engine and session maker in `app/database.py`.
- [ ] **Task 1.1.4:** Create `BaseModel` mixin in `app/models/base.py` providing UUIDv7 primary keys, `created_at`, `updated_at`, and soft-delete columns.
- [ ] **Task 1.1.5:** Create standard API envelope schemas (`ResponseEnvelope`, `ErrorEnvelope`, `PaginatedData`) in `app/schemas/common.py`.
- [ ] **Task 1.1.6:** Implement FastAPI app factory in `app/main.py` with global exception handlers and CORS middleware.
- [ ] **Task 1.1.7:** Create `/api/v1/health` endpoint for database connectivity verification.

#### Sprint 1.2: Core Models & Alembic Migration Foundation
- [ ] **Task 1.2.1:** Implement declarative models for `Organization`, `User`, `Role`, `Permission`, `RolePermission`, `UserRole`, `RefreshToken`, and `Invitation`.
- [ ] **Task 1.2.2:** Configure `alembic/env.py` for async migration generation targeting metadata.
- [ ] **Task 1.2.3:** Generate initial Alembic revision script: `0001_initial_auth_and_org_schema.py`.
- [ ] **Task 1.2.4:** Write system role seed script (`super_admin`, `org_admin`, `sales_manager`, `sales_rep`) and core permission entries.

---

### Phase 2: Authentication & Multi-Tenancy (Weeks 2–3)

#### Sprint 2.1: Authentication Flow & Token Lifecycle
- [ ] **Task 2.1.1:** Implement Argon2id password hashing and verification in `app/core/security.py`.
- [ ] **Task 2.1.2:** Implement JWT creation, verification, and decoding with access token (15m) and refresh token (7d) lifecycles.
- [ ] **Task 2.1.3:** Implement `POST /api/v1/auth/register` (creates tenant organization + initial admin user + default pipeline).
- [ ] **Task 2.1.4:** Implement `POST /api/v1/auth/login` (verifies credentials, issues tokens, resets failed attempts).
- [ ] **Task 2.1.5:** Implement `POST /api/v1/auth/refresh` with single-use refresh token rotation and revocation tracking.
- [ ] **Task 2.1.6:** Implement `POST /api/v1/auth/logout` and `GET /api/v1/auth/me`.

#### Sprint 2.2: Multi-Tenancy Enforcement & User Management
- [ ] **Task 2.2.1:** Implement `BaseRepository` in `app/repositories/base.py` enforcing automatic `organization_id` query scoping.
- [ ] **Task 2.2.2:** Implement Tenant Context Middleware in `app/middleware/tenant.py` to set PostgreSQL session variable `app.current_tenant`.
- [ ] **Task 2.2.3:** Implement user management endpoints: `GET /api/v1/users`, `POST /api/v1/users/invite`, `POST /api/v1/users/accept-invitation`.
- [ ] **Task 2.2.4:** Implement RBAC permission verification dependency (`require_permission("leads:create")`).
- [ ] **Task 2.2.5:** Author multi-tenant test suite in `tests/test_tenancy_isolation.py` ensuring Tenant A cannot read Tenant B's users.

---

### Phase 3: Core CRM — Leads, Contacts & Companies (Weeks 3–5)

#### Sprint 3.1: Leads Module
- [ ] **Task 3.1.1:** Create `Lead` model in `app/models/lead.py` and generate Alembic migration.
- [ ] **Task 3.1.2:** Build `LeadRepository` supporting status filters, owner filtering, full-text search, and pagination.
- [ ] **Task 3.1.3:** Build `LeadService` with duplicate email detection and status transition validation.
- [ ] **Task 3.1.4:** Implement endpoints: `GET /api/v1/leads`, `POST /api/v1/leads`, `GET /api/v1/leads/{id}`, `PATCH /api/v1/leads/{id}`, `DELETE /api/v1/leads/{id}`.

#### Sprint 3.2: Contacts & Companies
- [ ] **Task 3.2.1:** Create `Company` and `Contact` models with foreign key associations.
- [ ] **Task 3.2.2:** Build repositories and services for Company and Contact management.
- [ ] **Task 3.2.3:** Implement endpoints for Contacts and Companies CRUD with parent-child relationship expansion.

#### Sprint 3.3: Transactional Lead-to-Deal Conversion
- [ ] **Task 3.3.1:** Implement atomic conversion orchestration in `LeadService.convert_lead()` inside `async with session.begin():`.
- [ ] **Task 3.3.2:** Enforce idempotency: return `400 Bad Request` if lead is already converted.
- [ ] **Task 3.3.3:** Implement endpoint `POST /api/v1/leads/{id}/convert`.
- [ ] **Task 3.3.4:** Write exhaustive conversion integration tests:
  - Test A: Full conversion (Contact + Company + Deal).
  - Test B: Partial conversion (Contact only).
  - Test C: Transaction rollback on validation error (ensuring zero orphaned records).
  - Test D: Idempotency enforcement on repeat conversion attempts.

---

### Phase 4: Sales Pipeline & Deal Management (Weeks 5–7)

#### Sprint 4.1: Pipelines & Stages
- [ ] **Task 4.1.1:** Create `Pipeline` and `DealStage` models.
- [ ] **Task 4.1.2:** Build pipeline CRUD and stage reordering endpoint (`PUT /api/v1/pipelines/{id}/stages/reorder`).
- [ ] **Task 4.1.3:** Attach default pipeline creation hook into tenant registration.

#### Sprint 4.2: Deals Module & Kanban Operations
- [ ] **Task 4.2.1:** Create `Deal` model with `amount`, `probability`, `expected_close_date`, and stage references.
- [ ] **Task 4.2.2:** Implement deal stage transition logic in `DealService.update_stage()`:
  - Automatically set probability to stage default.
  - Require `lost_reason` when entering lost stage.
  - Set `actual_close_date = CURRENT_DATE` on won or lost.
- [ ] **Task 4.2.3:** Implement endpoints: `GET /api/v1/deals`, `POST /api/v1/deals`, `PATCH /api/v1/deals/{id}/stage`.

---

### Phase 5: Activities, Tasks & Productivity (Weeks 7–8)

- [ ] **Task 5.1:** Create `Activity`, `Task`, and `Note` models with polymorphic entity associations (`related_entity_type`, `related_entity_id`).
- [ ] **Task 5.2:** Build `ActivityService` logging calls, meetings, notes, and emails.
- [ ] **Task 5.3:** Build `TaskService` handling assignments, status progressions, and overdue detection.
- [ ] **Task 5.4:** Implement aggregate timeline endpoint: `GET /api/v1/{entity_type}/{entity_id}/timeline` combining notes, activities, and audit events.

---

### Phase 6: Supporting Modules & Extensibility (Weeks 8–10)

- [ ] **Task 6.1:** Implement Tags module (`tags`, `entity_tags`) and endpoints.
- [ ] **Task 6.2:** Implement Custom Fields system (`custom_fields`, `custom_field_values`) with dynamic validation against configured data types.
- [ ] **Task 6.3:** Implement File Attachments system with local storage driver and signed URL downloads.
- [ ] **Task 6.4:** Implement In-App Notifications system (`notifications` table + unread badge counters).
- [ ] **Task 6.5:** Implement Append-Only Audit Logging (`audit_logs`) recording high-value business transitions.

---

### Phase 7: Analytics, Reports & Automation (Weeks 10–12)

- [ ] **Task 7.1:** Build `ReportService` queries calculating:
  - Total revenue won vs lost.
  - Funnel conversion drop-off by stage.
  - Sales rep performance metrics.
- [ ] **Task 7.2:** Implement `GET /api/v1/reports/dashboard` and analytical endpoints.
- [ ] **Task 7.3:** Implement Rule-Based Automation Engine (`workflow_rules`, `workflow_executions`) evaluating domain events and triggering automated tasks and notifications.

---

### Phase 8: Frontend Presentation Layer (Weeks 6–12 Parallel)

- [ ] **Task F1 (Foundation):** Setup Next.js 14 App Router, Tailwind CSS, Lucide Icons, and Axios client with auto-refresh token interceptors.
- [ ] **Task F2 (Auth Flow):** Build `/login` and `/register` views with client-side form validation and Zustand session storage.
- [ ] **Task F3 (App Shell):** Build responsive layout shell with Collapsible Sidebar, Header, Breadcrumbs, and User Profile menu.
- [ ] **Task F4 (Leads & CRM):** Build Leads Table with search, status filters, pagination, Lead Details page, and Lead Conversion Modal dialog.
- [ ] **Task F5 (Sales Kanban):** Build Deals Pipeline View with drag-and-drop stage movement (using `@hello-pangea/dnd` or `@dnd-kit`).
- [ ] **Task F6 (Dashboard & Reports):** Build Dashboard UI with revenue KPI metric cards, funnel charts, and upcoming task reminders.

---

## 3. Testing Strategy & Quality Gates

### 3.1 Test Matrix
1. **Unit Tests:** Fast, isolated testing of business rules (e.g. password validator, weighted value calculations, permission evaluator).
2. **Integration Tests:** Execute against a real PostgreSQL test database:
   - Registration and JWT token rotation.
   - Cross-tenant data isolation (verifying Tenant A cannot read or mutate Tenant B's records).
   - Atomic lead conversion and transaction rollback.
   - Soft-delete exclusion from standard queries.
3. **API Contract Tests:** Verify response JSON adheres to standard `ResponseEnvelope` and `ErrorEnvelope`.

### 3.2 Definition of Done (DoD) per Module
- All endpoints defined in `NEXCRM_API_SPECIFICATION.md` are implemented.
- Automatic tenant scoping (`organization_id`) verified via tests.
- Input validation handled via Pydantic schemas returning `422` on invalid data.
- Unauthorized actions return `401`, forbidden actions return `403`.
- Integration tests pass with >= 80% coverage on domain services.

---

## 4. Next Step: Execution Plan for Milestone 1

We are ready to begin implementation of **Milestone 1**:
1. Create backend project layout and configuration (`pyproject.toml`, `requirements.txt`, `app/config.py`).
2. Establish database connectivity and base models (`app/database.py`, `app/models/base.py`).
3. Build the core tenancy and auth models (`Organization`, `User`, `Role`, `Permission`, `RefreshToken`).
4. Generate initial Alembic migration and seed system roles.
5. Implement authentication service and endpoints (`/auth/register`, `/auth/login`, `/auth/me`).
6. Verify multi-tenant isolation with automated tests.
