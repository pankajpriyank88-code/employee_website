# Employee Portal

An internal employee self-service portal with role-based access control. Employees manage their own attendance, leave, documents, and requests; HR/Admin manage all employee data and approvals. Every change to employee master data flows through an approval workflow and is audit-logged.

## Overview

The system is built around a strict access model: employees can view their own data and perform self-service actions on their own records, but can never edit their master data directly or touch another employee's data. All edits to official records go through a request → HR/Admin review → approve/reject cycle.

## Roles

| Role | Scope |
|------|-------|
| Employee | Own records only: view profile, apply leave, check in/out, download own payslip/ID card, upload own documents, raise tickets, submit resignation, give feedback |
| HR / Admin | All employee records: full CRUD, approve/reject leave and resignation, upload payslips, verify documents, manage announcements/policies/holidays, run performance cycles |
| Manager (optional) | Direct reports only: approve team leave, view team attendance, conduct performance reviews |

## Features

- Employee login (email + password) with password reset and security settings
- Employee dashboard and profile
- Attendance check-in / check-out with monthly summaries and history
- Leave application and multi-stage approval
- Salary / payslip download
- Company announcements and notice board
- Holiday calendar
- Task and project management
- Performance reviews
- Training and learning materials
- Company policies and documents
- ID card download
- Help desk / HR support tickets
- Employee directory (read-only, limited fields)
- Resignation requests
- Feedback and suggestions (optionally anonymous)
- Document upload (Aadhaar, PAN, bank details) with HR verification
- Change requests for master data, with audit logging

## Access Control

Access rules are enforced at the API layer, not just in the UI.

- Every request derives the acting user's `employee_id` from the authenticated session/JWT, never from a client-supplied parameter.
- Self-service queries are automatically filtered to `WHERE employee_id = current_user.employee_id`, so an employee cannot request another employee's records.
- Approval actions (approve/reject leave, resolve tickets, verify documents) are permitted only for HR/Admin or Manager roles, checked server-side on every request.
- Employees have read-only access to their own master data. Corrections (name, address, bank details, etc.) are submitted as change requests and applied only after HR/Admin approval.
- The employee directory exposes a restricted field set (name, designation, department, contact) and never salary, bank details, or documents.

## Tech Stack

- Frontend: React (or Next.js) — responsive SPA/PWA
- Backend: Node.js (Express/NestJS) or Django/Spring Boot — REST or GraphQL API
- Database: PostgreSQL
- File storage: S3-compatible object storage, encrypted at rest for sensitive documents (Aadhaar/PAN/bank, payslips, ID cards)
- Auth: JWT-based sessions with role claims; 2FA recommended for HR/Admin accounts

## Data Model

Core tables:

- `users` — id, email, password_hash, role, status
- `employees` — id, user_id (FK), name, DOB, designation, department, manager_id, joining_date, employment_status
- `attendance` — id, employee_id (FK), date, check_in_time, check_out_time, status
- `leave_applications` — id, employee_id (FK), leave_type, from_date, to_date, reason, status, approved_by
- `payslips` — id, employee_id (FK), month, year, file_url, generated_by
- `documents` — id, employee_id (FK), doc_type, file_url, verification_status
- `announcements` — id, title, content, posted_by, visible_to
- `holidays` — id, date, name, applicable_locations
- `tasks_projects` — id, project_name, assigned_to, deadline, status
- `performance_reviews` — id, employee_id (FK), reviewer_id (FK), cycle, rating, comments
- `training_materials` — id, title, file_url, category
- `policies_documents` — id, title, file_url, category
- `helpdesk_tickets` — id, employee_id (FK), category, description, status, assigned_to
- `resignation_requests` — id, employee_id (FK), notice_date, last_working_day, status, approved_by
- `feedback` — id, employee_id (nullable), message, category
- `change_requests` — id, employee_id (FK), field_name, old_value, new_value, status, reviewed_by
- `audit_log` — id, actor_id (FK), action, entity, entity_id, timestamp

`change_requests` and `audit_log` are the tables that enforce and record the access-control model: employee-initiated edits become change requests, and every HR/Admin action against employee data is written to the audit log.

## Getting Started

> Placeholder commands — adjust to match the stack you actually build with.

```bash
# Clone the repository
git clone <repository-url>
cd employee-portal

# Install dependencies
npm install            # frontend
# and your backend equivalent

# Configure environment
cp .env.example .env   # then fill in the values below

# Run database migrations
npm run migrate

# Start the app
npm run dev
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Secret for signing session tokens |
| `STORAGE_BUCKET` | S3-compatible bucket for documents and payslips |
| `STORAGE_ACCESS_KEY` | Object storage access key |
| `STORAGE_SECRET_KEY` | Object storage secret key |
| `SMTP_URL` | Mail server for notifications (approvals, resets) |

## Project Structure

> Adjust to your actual layout.

```
employee-portal/
├── frontend/          # React/Next.js app
├── backend/           # API server
│   ├── routes/        # API endpoints per module
│   ├── middleware/    # auth, RBAC, row-level scoping
│   ├── models/        # database models
│   └── services/      # business logic (approvals, notifications)
├── migrations/        # database schema migrations
└── docs/              # system design document, ERD
```

## Documentation

See `Employee_Portal_System_Design.docx` for the full system design, architecture diagram, ERD, access-control workflow, and module-wise access matrix.

## Security Notes

- Enforce all access rules server-side; treat the UI as untrusted.
- Encrypt sensitive documents at rest and log every access.
- Log every HR/Admin edit to employee data (who changed what, when).
- Require 2FA for accounts with cross-employee access.

## License

Proprietary — internal use only.
