---
ai-first: true
type: project
date: 2026-05-28
tags: [client-site, crm, nextjs, supabase, archived]
status: complete
---

# Anderton & Associates

## For future Claude
Completed client site for Anderton & Associates (web design agency). Full admin CRM + public contact form. Site is live and deployed — load this if Anderton asks for changes or if referencing the CRM/admin pattern used here.

---

## Identity
- **Client:** Anderton & Associates (web design agency)
- **Path:** `/Users/drive/anderton-associates`
- **Live URL:** anderton-associates-e08dbtf10-adobetoby-5572s-projects.vercel.app
- **Supabase:** `qnrkifdbkcbacgznoabs` — tables: clients, projects, contact_submissions
- **Admin URL:** `/admin/login`

## Features Built
- Admin auth: HMAC cookie `aa_admin_session` via `proxy.ts`
- Clients: list, detail, create, edit, delete
- Projects: list, detail, create, edit, delete (linked to clients)
- Contact form: submits to Supabase `contact_submissions` table
- Inquiries page: `/admin/inquiries` with read/unread toggle
- Dashboard: 4 stat cards (clients, projects, active projects, new inquiries)

## Auth Pattern
- `proxy.ts` — route protection; redirects `/admin/*` to login if no valid session
- `app/api/auth/route.ts` — POST login, DELETE logout
- `app/api/admin/users/route.ts` — user CRUD (super_admin only)
- Passwords stored as scrypt hashes, never plaintext
- Credentials stored in vault at manage.worker-bee.app (not here)

## Stack
Next.js 16, TypeScript, Tailwind CSS, Supabase (same project as jrs-auto-repair)
