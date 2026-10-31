# NexCRM — RESTful API Specification

**API Version:** `v1`  
**Base Path:** `/api/v1`  
**Protocol:** HTTPS / JSON REST  
**Authentication:** Bearer JWT in `Authorization` Header

---

## 1. Global API Standards & Conventions

### 1.1 Headers
- **`Authorization`**: `Bearer <access_token>` (Required for all protected endpoints).
- **`Content-Type`**: `application/json` (or `multipart/form-data` for file uploads).
- **`Accept`**: `application/json`

### 1.2 Unified Success Envelope
All `2xx` responses adhere to the standard success envelope:
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation completed successfully"
}
```

### 1.3 Unified Error Envelope
All `4xx` and `5xx` responses adhere to the standard error envelope:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Request validation failed.",
    "details": [
      {
        "field": "email",
        "message": "value is not a valid email address"
      }
    ]
  }
}
```

### 1.4 Pagination Standard
List endpoints accept standard pagination query parameters:
- `page`: integer (default: `1`, minimum: `1`)
- `page_size`: integer (default: `20`, maximum: `100`)

Paginated responses structure the `data` object as:
```json
{
  "success": true,
  "data": {
    "items": [ ... ],
    "pagination": {
      "total": 142,
      "page": 1,
      "page_size": 20,
      "total_pages": 8,
      "has_next": true,
      "has_prev": false
    }
  },
  "message": "Records retrieved successfully"
}
```

### 1.5 Sorting & Filtering
- Sorting: `?sort_by=<field_name>&sort_order=asc|desc` (default: `sort_by=created_at&sort_order=desc`).
- Search: `?search=<text>` (performs full-text keyword search across indexed fields).
- Filtering: Direct query parameters (e.g. `?status=qualified&owner_id=018f...`).

---

## 2. Authentication Module (`/api/v1/auth`)

### 2.1 Register Tenant & Initial Admin User
Creates a new tenant organization, assigns the initial Super/Org Admin user, seeds the default pipeline, and issues authentication tokens.

- **Method / Path:** `POST /api/v1/auth/register`
- **Access:** Public
- **Request Body:**
```json
{
  "organization_name": "Acme Innovations Ltd",
  "organization_slug": "acme-innovations",
  "first_name": "Sarah",
  "last_name": "Connor",
  "email": "sarah.connor@acme.com",
  "password": "StrongPassword@2026"
}
```
- **Response `201 Created`:**
```json
{
  "success": true,
  "data": {
    "organization": {
      "id": "018f3a50-7f2e-7000-8800-000000000001",
      "name": "Acme Innovations Ltd",
      "slug": "acme-innovations",
      "status": "trial"
    },
    "user": {
      "id": "018f3a50-7f2e-7000-8800-000000000002",
      "email": "sarah.connor@acme.com",
      "first_name": "Sarah",
      "last_name": "Connor",
      "roles": ["org_admin"]
    },
    "tokens": {
      "access_token": "eyJhbGciOiJIUzI1...",
      "refresh_token": "9f83c1b4e...",
      "token_type": "Bearer",
      "expires_in": 900
    }
  },
  "message": "Organization and administrator account created successfully."
}
```
- **Error Codes:** `409 Conflict` (`SLUG_ALREADY_EXISTS`, `EMAIL_ALREADY_REGISTERED`), `422 Unprocessable Entity`.

---

### 2.2 Authenticate / Login
- **Method / Path:** `POST /api/v1/auth/login`
- **Access:** Public
- **Request Body:**
```json
{
  "email": "sarah.connor@acme.com",
  "password": "StrongPassword@2026"
}
```
- **Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "018f3a50-7f2e-7000-8800-000000000002",
      "organization_id": "018f3a50-7f2e-7000-8800-000000000001",
      "email": "sarah.connor@acme.com",
      "first_name": "Sarah",
      "last_name": "Connor",
      "roles": ["org_admin"]
    },
    "tokens": {
      "access_token": "eyJhbGciOiJIUz...",
      "refresh_token": "82bc394d...",
      "token_type": "Bearer",
      "expires_in": 900
    }
  },
  "message": "Login successful"
}
```
- **Error Codes:** `401 Unauthorized` (`INVALID_CREDENTIALS`, `ACCOUNT_LOCKED`, `ORGANIZATION_SUSPENDED`).

---

### 2.3 Rotate Refresh Token
- **Method / Path:** `POST /api/v1/auth/refresh`
- **Access:** Public
- **Request Body:**
```json
{
  "refresh_token": "82bc394d..."
}
```
- **Response `200 OK`:** Returns fresh `access_token` and rotated `refresh_token`.
- **Error Codes:** `401 Unauthorized` (`INVALID_REFRESH_TOKEN`, `TOKEN_REVOKED`).

---

### 2.4 Logout
- **Method / Path:** `POST /api/v1/auth/logout`
- **Access:** Authenticated
- **Request Body:**
```json
{
  "refresh_token": "82bc394d..."
}
```
- **Response `200 OK`:**
```json
{
  "success": true,
  "data": null,
  "message": "Logged out successfully"
}
```

---

### 2.5 Current User Profile
- **Method / Path:** `GET /api/v1/auth/me`
- **Access:** Authenticated
- **Response `200 OK`:** Returns current user details, organization summary, and active permissions list.

---

## 3. Organizations Module (`/api/v1/organizations`)

### 3.1 Get Current Tenant Organization
- **Method / Path:** `GET /api/v1/organizations/me`
- **Required Permission:** `organization:view`
- **Response `200 OK`:** Returns tenant settings, business defaults, plan, and status.

### 3.2 Update Tenant Organization Settings
- **Method / Path:** `PATCH /api/v1/organizations/me`
- **Required Permission:** `organization:update`
- **Request Body:**
```json
{
  "name": "Acme Global Solutions",
  "domain": "acme.com",
  "settings": {
    "currency": "USD",
    "timezone": "America/New_York",
    "fiscal_year_start_month": 1
  }
}
```
- **Response `200 OK`:** Updated organization payload.

---

## 4. User Management & Invitations (`/api/v1/users`)

### 4.1 List Users
- **Method / Path:** `GET /api/v1/users`
- **Query Params:** `page`, `page_size`, `role_id`, `is_active`, `search`
- **Required Permission:** `users:view`

### 4.2 Invite Team Member
- **Method / Path:** `POST /api/v1/users/invite`
- **Required Permission:** `users:create`
- **Request Body:**
```json
{
  "email": "alex.sales@acme.com",
  "role_id": "018f3a50-7f2e-7000-8800-000000000010"
}
```
- **Response `201 Created`:**
```json
{
  "success": true,
  "data": {
    "invitation_id": "018f3a50-7f2e-7000-8800-000000000099",
    "email": "alex.sales@acme.com",
    "status": "pending",
    "expires_at": "2026-11-03T11:00:00Z"
  },
  "message": "Invitation dispatched successfully."
}
```

### 4.3 Accept Team Invitation
- **Method / Path:** `POST /api/v1/users/accept-invitation`
- **Access:** Public
- **Request Body:**
```json
{
  "token": "raw_invitation_secret_token",
  "first_name": "Alex",
  "last_name": "Vance",
  "password": "SecurePassword123!"
}
```
- **Response `200 OK`:** Returns user entity and initial JWT token pair.

### 4.4 Update / Deactivate User
- `PATCH /api/v1/users/{id}` (Update profile or role assignment)
- `DELETE /api/v1/users/{id}` (Soft deletes/deactivates user; requires reassigning active leads/deals)

---

## 5. Roles & RBAC (`/api/v1/roles`, `/api/v1/permissions`)

### 5.1 List Roles
- **Method / Path:** `GET /api/v1/roles`
- **Required Permission:** `roles:view`
- **Response `200 OK`:** List of standard system roles (`super_admin`, `org_admin`, `sales_manager`, `sales_rep`) and custom tenant roles.

### 5.2 Create Custom Role
- **Method / Path:** `POST /api/v1/roles`
- **Required Permission:** `roles:create`
- **Request Body:**
```json
{
  "name": "Junior Sales Agent",
  "slug": "junior_sales_agent",
  "description": "Can view own leads and deals but cannot export data",
  "permission_ids": [
    "018f3a50-7f2e-7000-8800-000000000101",
    "018f3a50-7f2e-7000-8800-000000000102"
  ]
}
```

### 5.3 List All Available System Permissions
- **Method / Path:** `GET /api/v1/permissions`
- **Required Permission:** `roles:view`

---

## 6. Leads Module (`/api/v1/leads`)

### 6.1 List Leads
- **Method / Path:** `GET /api/v1/leads`
- **Query Params:** `page`, `page_size`, `status`, `source`, `priority`, `owner_id`, `search`
- **Required Permission:** `leads:view`

### 6.2 Create Lead
- **Method / Path:** `POST /api/v1/leads`
- **Required Permission:** `leads:create`
- **Request Body:**
```json
{
  "first_name": "Michael",
  "last_name": "Scott",
  "email": "mscott@dundermifflin.com",
  "phone": "+1-570-555-0133",
  "company_name": "Dunder Mifflin Paper Co.",
  "title": "Regional Manager",
  "source": "referral",
  "priority": "high",
  "owner_id": "018f3a50-7f2e-7000-8800-000000000002",
  "notes": "Interested in paper automation workflows"
}
```
- **Response `201 Created`:** Created lead record with status `new`.

### 6.3 Get Lead Details
- **Method / Path:** `GET /api/v1/leads/{id}`
- **Required Permission:** `leads:view`

### 6.4 Update Lead
- **Method / Path:** `PATCH /api/v1/leads/{id}`
- **Required Permission:** `leads:update`

### 6.5 Delete Lead (Soft Delete)
- **Method / Path:** `DELETE /api/v1/leads/{id}`
- **Required Permission:** `leads:delete`

### 6.6 Lead Conversion (Transactional Business Operation)
Converts a qualified lead into associated Contact, Company, and Deal entities in an atomic database transaction.

- **Method / Path:** `POST /api/v1/leads/{id}/convert`
- **Required Permission:** `leads:convert`
- **Request Body:**
```json
{
  "create_contact": true,
  "contact": {
    "first_name": "Michael",
    "last_name": "Scott",
    "email": "mscott@dundermifflin.com",
    "phone": "+1-570-555-0133",
    "title": "Regional Manager"
  },
  "create_company": true,
  "company": {
    "name": "Dunder Mifflin Paper Co.",
    "website": "https://dundermifflin.com",
    "industry": "Paper & Forestry"
  },
  "create_deal": true,
  "deal": {
    "title": "Dunder Mifflin Enterprise License",
    "amount": 48000.00,
    "pipeline_id": "018f3a50-7f2e-7000-8800-000000000300",
    "stage_id": "018f3a50-7f2e-7000-8800-000000000301",
    "expected_close_date": "2026-12-15"
  }
}
```
- **Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "lead_id": "018f3a50-7f2e-7000-8800-000000000201",
    "status": "converted",
    "converted_at": "2026-10-31T12:00:00Z",
    "contact": {
      "id": "018f3a50-7f2e-7000-8800-000000000401",
      "first_name": "Michael",
      "last_name": "Scott",
      "email": "mscott@dundermifflin.com"
    },
    "company": {
      "id": "018f3a50-7f2e-7000-8800-000000000501",
      "name": "Dunder Mifflin Paper Co."
    },
    "deal": {
      "id": "018f3a50-7f2e-7000-8800-000000000601",
      "title": "Dunder Mifflin Enterprise License",
      "amount": 48000.00
    }
  },
  "message": "Lead converted successfully."
}
```
- **Error Codes:** `400 Bad Request` (`LEAD_ALREADY_CONVERTED`, `INVALID_CONVERSION_STATE`), `404 Not Found`.

---

## 7. Contacts & Companies (`/api/v1/contacts`, `/api/v1/companies`)

### 7.1 Contacts CRUD
- `GET /api/v1/contacts` — Paginated contacts list (filterable by company_id, owner_id, search).
- `POST /api/v1/contacts` — Create contact record.
- `GET /api/v1/contacts/{id}` — Get contact details including company, deals, and timeline.
- `PATCH /api/v1/contacts/{id}` — Update contact details.
- `DELETE /api/v1/contacts/{id}` — Soft delete contact.

### 7.2 Companies CRUD
- `GET /api/v1/companies` — Paginated companies list.
- `POST /api/v1/companies` — Create company account.
- `GET /api/v1/companies/{id}` — Get company details, associated contacts, and active deals.
- `PATCH /api/v1/companies/{id}` — Update company account.
- `DELETE /api/v1/companies/{id}` — Soft delete company.

---

## 8. Deals & Pipelines (`/api/v1/deals`, `/api/v1/pipelines`)

### 8.1 Deals CRUD & Stage Movement
- `GET /api/v1/deals` — List deals (filterable by pipeline_id, stage_id, owner_id, close_date range).
- `POST /api/v1/deals` — Create deal.
- `GET /api/v1/deals/{id}` — Get deal details.
- `PATCH /api/v1/deals/{id}` — Update deal metadata.
- `DELETE /api/v1/deals/{id}` — Soft delete deal.

#### Transition Deal Stage (Kanban Drag-and-Drop)
- **Method / Path:** `PATCH /api/v1/deals/{id}/stage`
- **Required Permission:** `deals:update`
- **Request Body:**
```json
{
  "stage_id": "018f3a50-7f2e-7000-8800-000000000305",
  "lost_reason": null,
  "won_reason": "Customer agreed to 2-year upfront commitment"
}
```
- **Response `200 OK`:** Updated deal with updated stage, probability, and calculated weighted amount.

### 8.2 Pipelines & Stages Management
- `GET /api/v1/pipelines` — List pipelines with nested ordered stages and deal counts.
- `POST /api/v1/pipelines` — Create pipeline.
- `PATCH /api/v1/pipelines/{id}` — Update pipeline.
- `POST /api/v1/pipelines/{id}/stages` — Add stage to pipeline.
- `PUT /api/v1/pipelines/{id}/stages/reorder` — Bulk reorder positions of stages.

---

## 9. Activities, Tasks & Productivity (`/api/v1/activities`, `/api/v1/tasks`, `/api/v1/notes`)

### 9.1 Activities (Calls, Meetings, Emails)
- `GET /api/v1/activities` — Filterable by `type`, `due_at` range, `owner_id`, `related_entity_type`, `related_entity_id`.
- `POST /api/v1/activities` — Create activity:
```json
{
  "type": "meeting",
  "subject": "Platform Demo & Security Review",
  "due_at": "2026-11-05T15:00:00Z",
  "duration_minutes": 45,
  "location": "https://meet.google.com/xyz-abc",
  "related_entity_type": "deal",
  "related_entity_id": "018f3a50-7f2e-7000-8800-000000000601"
}
```
- `PATCH /api/v1/activities/{id}/complete` — Mark activity completed with outcome and notes.

### 9.2 Tasks
- `GET /api/v1/tasks` — List tasks (filterable by `status`, `assigned_to`, `priority`).
- `GET /api/v1/tasks/my` — Shortcut for current authenticated user's pending tasks.
- `POST /api/v1/tasks` — Create task.
- `PATCH /api/v1/tasks/{id}/complete` — Mark task status as completed.

### 9.3 Notes
- `GET /api/v1/notes?related_entity_type=deal&related_entity_id=...` — Retrieve entity notes.
- `POST /api/v1/notes` — Create note.
- `PATCH /api/v1/notes/{id}` — Update note content or toggle pinned state.

---

## 10. Tags & Custom Fields (`/api/v1/tags`, `/api/v1/custom-fields`)

### 10.1 Tags Management
- `GET /api/v1/tags` — List tenant tags.
- `POST /api/v1/tags` — Create tag (`name`, `color`).
- `POST /api/v1/{entity_type}/{entity_id}/tags` — Attach tags to entity.
- `DELETE /api/v1/{entity_type}/{entity_id}/tags/{tag_id}` — Detach tag from entity.

### 10.2 Custom Fields Schema Definition
- `GET /api/v1/custom-fields?entity_type=lead` — Retrieve configured custom fields for entity.
- `POST /api/v1/custom-fields` — Register a new dynamic field definition:
```json
{
  "entity_type": "lead",
  "field_name": "preferred_cloud_provider",
  "field_label": "Preferred Cloud Provider",
  "field_type": "select",
  "options": ["AWS", "Google Cloud", "Microsoft Azure", "On-Premises"],
  "is_required": false,
  "position": 5
}
```

---

## 11. Workflow Automation Engine (`/api/v1/workflows`)

### 11.1 List & Manage Workflows
- `GET /api/v1/workflows` — List configured automation rules.
- `POST /api/v1/workflows` — Create workflow rule:
```json
{
  "name": "Auto-Create Proposal Follow-Up Task",
  "entity_type": "deal",
  "trigger_event": "stage_changed",
  "conditions": [
    {
      "field": "stage.name",
      "operator": "equals",
      "value": "Proposal"
    }
  ],
  "actions": [
    {
      "type": "create_task",
      "parameters": {
        "title": "Prepare & Send Formal SOW",
        "due_in_days": 2,
        "priority": "high",
        "assign_to": "deal_owner"
      }
    }
  ],
  "is_active": true
}
```
- `PATCH /api/v1/workflows/{id}/toggle` — Enable or disable workflow.
- `GET /api/v1/workflows/{id}/executions` — View execution history and diagnostic logs.

---

## 12. Reports & Analytics (`/api/v1/reports`)

### 12.1 Executive Dashboard Metrics
- **Method / Path:** `GET /api/v1/reports/dashboard`
- **Required Permission:** `reports:view`
- **Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "total_revenue_won": 412500.00,
    "pipeline_weighted_value": 894000.00,
    "active_deals_count": 34,
    "lead_conversion_rate": 28.4,
    "revenue_trend": [
      { "month": "2026-08", "revenue": 115000.00 },
      { "month": "2026-09", "revenue": 142000.00 },
      { "month": "2026-10", "revenue": 155500.00 }
    ],
    "deals_by_stage": [
      { "stage_name": "Discovery", "count": 12, "value": 240000.00 },
      { "stage_name": "Proposal", "count": 8, "value": 310000.00 },
      { "stage_name": "Negotiation", "count": 6, "value": 220000.00 }
    ]
  },
  "message": "Dashboard metrics loaded successfully"
}
```

### 12.2 Specialized Analytical Endpoints
- `GET /api/v1/reports/sales` — Revenue breakdown by sales rep, pipeline, and date range.
- `GET /api/v1/reports/leads` — Lead volume by acquisition source and qualification funnel drop-off.
- `GET /api/v1/reports/forecast` — Forecasted revenue by projected close quarters.

---

## 13. Notifications, Attachments & Audit Logs

### 13.1 Notifications (`/api/v1/notifications`)
- `GET /api/v1/notifications` — Paginated user alerts.
- `GET /api/v1/notifications/unread-count` — Count of unread alerts.
- `PATCH /api/v1/notifications/{id}/read` — Mark alert read.
- `POST /api/v1/notifications/read-all` — Mark all alerts read.

### 13.2 File Attachments (`/api/v1/attachments`)
- `POST /api/v1/attachments` — `multipart/form-data` upload (`file`, `related_entity_type`, `related_entity_id`).
- `GET /api/v1/attachments/{id}/download` — Get secure signed download URL or binary stream.
- `DELETE /api/v1/attachments/{id}` — Remove file and delete from storage backend.

### 13.3 Audit Trail (`/api/v1/audit-logs`)
- **Method / Path:** `GET /api/v1/audit-logs`
- **Required Permission:** `audit_logs:view`
- **Query Params:** `entity_type`, `entity_id`, `user_id`, `action`, `start_date`, `end_date`
