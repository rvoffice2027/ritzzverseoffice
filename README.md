# Ritzzverse India — Internal Office Workspace

Private employee operations application built with React + Vite + Supabase, deployable on Vercel. Includes password login, role-based access, employee task assignment, priority/deadlines, scheduling, poster/video submissions, approval workflow, leave requests/counter, clock-in/out, client projects and team directory.

## 1. Set up Supabase
1. Create a project at https://supabase.com/dashboard.
2. Open **SQL Editor**, paste `supabase/schema.sql`, and run it once. Do not run it again after data exists.
3. In **Authentication > Providers**, keep Email enabled and disable public sign-ups in **Authentication > Settings** (the exact location may vary). Do not enable anonymous sign-in.
4. Create employee accounts via **Authentication > Users > Add user** (invite email is recommended). **Never share the Supabase service-role key** in frontend code or Vercel variables.
5. For every user, use SQL Editor to add a matching profile, replacing the UUID with the actual Authentication user ID:

```sql
insert into public.profiles(id,full_name,email,role,annual_leave_allowance)
values ('USER_UUID_FROM_AUTH','Employee Name','employee@your-company-domain.com','employee',18);
```

6. For the first admin use the same statement but role `admin`. Only an existing administrator of your Supabase project should run this SQL. Do not allow employees to access the SQL Editor. There is intentionally no self-service sign-up or frontend role-editing interface.
7. To restrict to company email accounts, invite **only addresses belonging to your verified company domain**. Supabase's built-in email/password login does not by itself enforce a company domain; enforce that policy in your admin onboarding process or implement an auth hook/SSO for strict domain enforcement. A custom domain is **not required** to test with invited addresses.

## 2. Run locally
```bash
npm install
cp .env.example .env.local
npm run dev
```
On Windows PowerShell use `Copy-Item .env.example .env.local` instead of `cp` if necessary. Fill in `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` from Supabase **Project Settings > API**. The anon/publishable key is intended for frontend use; the service-role/secret key is not.

## 3. GitHub + Vercel
1. Create a private GitHub repository and upload all project files **except** `.env.local` and `node_modules`.
2. Import the repository at https://vercel.com/new.
3. Framework: **Vite**. Build command: `npm run build`. Output directory: `dist`.
4. Add the two `VITE_` variables in Vercel Project Settings > Environment Variables.
5. Deploy. On every push to the main branch, Vercel redeploys automatically.

## 4. Important operational notes
- This is a working starter app, **not a production-audited HR/payroll system**. Test authorization and all workflows with separate employee/manager accounts before real staff use.
- Leave allowance is a manually set annual entitlement; the counter currently counts **calendar days**, not working days/holidays, and does not automatically reset by leave year.
- Employee ratings/scoring are not yet implemented. Priority labels apply to tasks, not employee performance.
- Submission files are private in Supabase Storage and are opened via short-lived signed URLs. The upload limit is 50 MB; for larger video files, use resumable uploads or external cloud storage.
- No client delivery portal, email notifications, automatic scheduling, Google Workspace SSO, payroll, or audit-log module is included.
- Attendance stores timestamps in UTC and displays local browser time. The database `current_date` check uses database timezone; for India operations, test around midnight and adjust to Asia/Kolkata if needed.
- Profiles are provisioned manually in Supabase for security. The team page is read-only.
- Admin/manager role changes and employee onboarding require Supabase dashboard access.
- Always use strong passwords, enable MFA where available, and verify RLS policies before handling sensitive employee data.
