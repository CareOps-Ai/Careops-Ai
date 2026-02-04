# CareOps AI - Production SaaS Platform

Complete case management system for adult care providers (245D, waiver services, adult foster care) with AI assistance, compliance tracking, and audit-ready documentation.

## 🎯 Project Status

**Current Phase**: Architecture & Foundation Setup
**Existing**: Frontend prototype (HTML/CSS/JS demo)
**Building**: Full production Next.js + Supabase backend

## 🏗️ Architecture Overview

### Tech Stack
- **Frontend**: Next.js 14 (App Router) + TypeScript + Tailwind CSS + shadcn/ui
- **Backend**: Next.js API Routes + Server Actions
- **Database**: PostgreSQL via Supabase
- **ORM**: Prisma
- **Auth**: Supabase Auth with RBAC
- **Storage**: Supabase Storage (documents, PDFs)
- **AI**: OpenAI API + pgvector for RAG
- **PDF**: pdf-lib for server-side generation
- **Email**: Resend (optional)

### Multi-Tenant Model
- Every record has `org_id` (organization/tenant)
- Row Level Security (RLS) enforces data isolation
- Users belong to one org with assigned role
- Strict RBAC at DB and application layer

### Security Model
- Supabase RLS policies for all tables
- Role-based permissions (ADMIN, SUPERVISOR, STAFF, CONTRACTOR, GUARDIAN, AUDITOR)
- Signed URLs for documents
- Audit logging for all critical actions
- Input validation with Zod
- No client-side secrets

## 👥 User Roles & Permissions

### ADMIN
- Full access within organization
- Manage users, assign roles
- Configure org settings
- Export audit packets
- Access all clients and data

### SUPERVISOR
- Manage staff assignments
- Review/approve documentation
- View team dashboards
- Access incidents and follow-ups
- Generate reports

### STAFF
- Create/edit notes for assigned clients
- Upload documents
- View assigned caseload only
- Submit incidents
- Use AI assistant

### CONTRACTOR
- Limited access to assigned clients only
- Cannot view org-wide data
- Create notes for assigned clients
- Upload required documents

### GUARDIAN/CLIENT
- Portal-only access
- View shared documents
- Sign forms electronically
- Upload documents (pending verification)
- No access to org data

### AUDITOR
- Read-only access
- View selected clients only
- Generate audit packets
- Export compliance reports
- No edit permissions

## 📦 Core Modules

### 1. Authentication & Onboarding
- Email/password via Supabase Auth
- Organization creation flow
- User invitation system
- Role assignment
- SSO support (future)

### 2. Dashboard
**Widgets**:
- Active clients count
- Notes due today
- Open incidents
- Pending signatures
- Audit readiness score
- Team activity feed (supervisor/admin)

### 3. Client Management
**Client Profile Tabs**:
- Overview (demographics, contacts, program)
- Documents (categorized, versioned)
- Service Plans (structured + attachments)
- Progress Notes (templates, approvals)
- Incidents (timeline, follow-ups)
- Portal Sharing (guardian access)

**Features**:
- Client search & filters
- Staff assignments
- Custom fields per program type
- Document version control
- Activity timeline

### 4. Documentation System
**Form Templates**:
- Daily Service Note
- Waiver Support Log
- Medication Error Report
- Adult Protection Incident
- Release of Information (ROI)

**Features**:
- JSON-based form definitions
- Dynamic UI rendering
- Required field validation
- Supervisor approval workflow
- Immutable audit trail
- Template versioning

### 5. Incident Management
- Incident types & severity levels
- Timeline tracking
- Required follow-up tasks
- Supervisor review queue
- Attachment support
- Overdue alerts
- State transitions (open → review → closed)

### 6. Guardian Portal (Traverse Connect)
- Secure external accounts
- Document sharing with permissions
- Electronic signature requests
- Document upload inbox
- Verification workflow
- Access logs

### 7. AI Compliance Assistant (RAG)
**Knowledge Base**:
- Upload policy documents (PDF, DOCX, TXT)
- Automatic chunking & embedding
- Vector search with pgvector
- Citation tracking

**Chat Features**:
- Policy Q&A with source citations
- "Suggest Note" feature
- Compliance checking
- Never invents policy
- Human review required

### 8. Audit Export System
**Generate DHS Audit Packet**:
- Client-specific or date range
- ZIP file containing:
  - PDF cover sheet
  - All notes (consolidated PDF)
  - Incidents timeline (PDF)
  - Signed forms (original PDFs)
  - Key documents
  - Training compliance snapshot

**Audit Logging**:
- All exports logged
- Who, what, when, where
- Immutable records

### 9. Staff Management
- User directory
- Invitation system
- Role management
- Deactivation workflow
- Timesheets (optional)
- Assignment tracking

### 10. Compliance & Audit Logs
**Logged Events**:
- Authentication (login, logout)
- Role changes
- Client create/update/delete
- Document upload/download/share
- Portal access grants
- Signature events
- Export generation
- Policy violations

**Log Structure**:
- Actor (user_id)
- Organization (org_id)
- Entity (type, id)
- Action
- Metadata (JSON)
- Timestamp
- IP address

## 🗄️ Database Schema

### Core Tables
```
organizations
  - id, name, settings, created_at

users
  - id (maps to Supabase auth)
  - email, name, avatar_url

memberships
  - user_id, org_id, role, created_at

clients
  - id, org_id, first_name, last_name, dob, program_type, status, assigned_staff[]

client_assignments
  - client_id, user_id, role, active, created_at

documents
  - id, org_id, client_id, category, file_url, version, uploaded_by, created_at

notes
  - id, org_id, client_id, template_type, json_payload, status, author_id, reviewer_id, created_at

incidents
  - id, org_id, client_id, incident_type, severity, description, follow_ups, status, created_at

signatures
  - id, org_id, client_id, document_id, signer_email, status, signed_at, ip_address

portal_users
  - id, email, client_id, relationship, created_at

portal_access_grants
  - portal_user_id, document_id, expires_at, created_at

uploads_inbox
  - id, portal_user_id, file_url, status, verified_by, verified_at

knowledge_base_documents
  - id, org_id, title, file_url, uploaded_by, processed_at

knowledge_base_chunks
  - id, document_id, content, embedding (vector), chunk_index

audit_logs
  - id, org_id, user_id, entity_type, entity_id, action, metadata, ip_address, created_at

time_logs
  - id, org_id, user_id, client_id, start_time, end_time, notes, approved
```

### RLS Policies
Each table has policies for:
- Tenant isolation (org_id match)
- Role-based access
- Assignment-based access (staff sees only assigned clients)
- Portal user isolation

## 🚀 Setup Instructions

### Prerequisites
- Node.js 18+
- PostgreSQL (or Supabase account)
- OpenAI API key
- Supabase project

### 1. Clone & Install
```bash
git clone <repo>
cd careops-ai
npm install
```

### 2. Environment Setup
```bash
cp .env.example .env.local
```

Required environment variables:
```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Database
DATABASE_URL=

# OpenAI
OPENAI_API_KEY=

# Email (optional)
RESEND_API_KEY=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 3. Database Setup
```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate dev

# Apply RLS policies
npm run setup:rls

# Seed demo data
npm run seed
```

### 4. Run Development Server
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

### 5. Demo Credentials
After seeding:
```
Admin:
  Email: admin@demo.org
  Password: DemoPass123!

Supervisor:
  Email: supervisor@demo.org
  Password: DemoPass123!

Staff:
  Email: staff@demo.org
  Password: DemoPass123!

Guardian Portal:
  Email: guardian@demo.org
  Password: DemoPass123!
```

## 📁 Project Structure

```
careops-ai/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth routes
│   │   ├── login/
│   │   ├── signup/
│   │   └── onboarding/
│   ├── (dashboard)/              # Protected routes
│   │   ├── dashboard/
│   │   ├── clients/
│   │   │   └── [id]/             # Client detail pages
│   │   ├── documents/
│   │   ├── incidents/
│   │   ├── ai/                   # AI Assistant
│   │   ├── audit/                # Audit logs & export
│   │   ├── staff/
│   │   └── settings/
│   ├── portal/                   # Guardian portal
│   └── api/                      # API routes
│       ├── auth/
│       ├── clients/
│       ├── documents/
│       ├── incidents/
│       ├── chat/                 # AI RAG chat
│       ├── audit/
│       └── webhooks/
├── components/
│   ├── ui/                       # shadcn/ui components
│   ├── dashboard/
│   ├── clients/
│   ├── forms/
│   ├── portal/
│   └── layout/
├── lib/
│   ├── auth/                     # Auth utilities
│   ├── db/                       # Prisma client
│   ├── supabase/                 # Supabase clients
│   ├── ai/                       # OpenAI + RAG
│   ├── pdf/                      # PDF generation
│   ├── storage/                  # File handling
│   └── utils/
├── prisma/
│   ├── schema.prisma             # Database schema
│   ├── migrations/
│   └── seed.ts
├── supabase/
│   ├── policies.sql              # RLS policies
│   └── functions/                # Edge functions (optional)
├── public/
├── types/
│   └── index.ts                  # TypeScript types
├── middleware.ts                 # Auth middleware
├── .env.example
├── .env.local
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

## 🔄 Development Workflow

### 1. Create Feature Branch
```bash
git checkout -b feature/incident-management
```

### 2. Database Changes
```bash
# Modify prisma/schema.prisma
npx prisma migrate dev --name add_incidents_table
```

### 3. Update RLS Policies
```bash
# Edit supabase/policies.sql
npm run setup:rls
```

### 4. Test & Commit
```bash
npm run test
git add .
git commit -m "Add incident management module"
```

### 5. Deploy
```bash
npm run build
# Deploy to Vercel/Railway/etc.
```

## 🧪 Testing Strategy

### Unit Tests
- Utils & helpers
- Form validation
- Permission checks
- PDF generation

### Integration Tests
- API routes
- Database operations
- RAG pipeline
- Audit export

### E2E Tests
- User flows
- Multi-tenant isolation
- Role permissions
- Portal workflows

## 📊 Monitoring & Observability

### Logs
- Application logs (Winston)
- Audit logs (database)
- Error tracking (Sentry)

### Metrics
- API response times
- Database query performance
- AI request latency
- Storage usage per org

### Alerts
- Failed auth attempts
- RLS policy violations
- Overdue incidents
- Storage limits

## 🚢 Deployment

### Recommended Platform: Vercel + Supabase

#### Vercel Setup
```bash
npm install -g vercel
vercel login
vercel --prod
```

#### Environment Variables
Configure in Vercel dashboard:
- All .env.local variables
- Production database URL
- API keys

#### Post-Deployment
1. Run migrations: `npm run db:migrate`
2. Apply RLS policies
3. Seed production data (optional)
4. Configure custom domain
5. Set up monitoring

### Alternative: Railway / Render / AWS

See deployment guides in `/docs/deployment/`

## 🔐 Security Checklist

- [ ] All tables have RLS policies
- [ ] Supabase service role key secured
- [ ] API rate limiting configured
- [ ] File uploads scanned
- [ ] CSP headers set
- [ ] CORS configured
- [ ] Input validation with Zod
- [ ] SQL injection prevented (Prisma)
- [ ] XSS protection
- [ ] Session management secure
- [ ] Audit logs immutable
- [ ] Secrets in env vars only
- [ ] HTTPS enforced
- [ ] Dependency scanning enabled

## 📚 Documentation

- [API Documentation](./docs/api.md)
- [Database Schema](./docs/database.md)
- [RLS Policies](./docs/rls.md)
- [AI RAG System](./docs/rag.md)
- [Audit Export](./docs/audit.md)
- [Portal Guide](./docs/portal.md)
- [Deployment](./docs/deployment.md)

## 🎯 Roadmap

### Phase 1: MVP (Current)
- ✅ Auth & RBAC
- ✅ Client management
- ✅ Documentation system
- ✅ Basic AI assistant
- ✅ Audit logs
- ✅ Assessment reminders

### Phase 2: Enhanced Features
- [ ] Advanced incident workflows
- [ ] Training compliance tracking
- [ ] Medication tracking
- [ ] Billing integration
- [ ] Mobile app (React Native)
- [ ] Offline mode

### Phase 3: Enterprise
- [ ] SSO (SAML, OAuth)
- [ ] Advanced analytics
- [ ] Custom reporting builder
- [ ] API for integrations
- [ ] White-label options
- [ ] Multi-language support

## 🤝 Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md)

## 📄 License

Proprietary - All rights reserved

## 🆘 Support

- Documentation: https://docs.careops.ai
- Email: support@careops.ai
- Slack: Join workspace

---

**Built with** ❤️ **for adult care providers**
