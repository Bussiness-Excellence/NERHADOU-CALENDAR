# Nerhadou Marketing Calendar

A shared calendar for the marketing team. Four groups, each with a manager who
approves their group's events. The super admin creates everyone's account.

Runs as one HTML file on GitHub Pages, with Supabase behind it for sign-in,
data, and live updates.

```
index.html                              the whole app
supabase/migrations/001_schema.sql      tables, security rules, triggers
supabase/functions/admin-users/         creates and manages user accounts
assets/                                 logo source files
```

---

## Setting it up

### 1. Database

Supabase dashboard → SQL Editor → New query → paste `001_schema.sql` → Run.
It's safe to run more than once.

### 2. Your own account

Authentication → Users → **Add user**. Enter your email and a password, and
tick **Auto Confirm User**.

Then back in the SQL editor, make yourself the super admin:

```sql
UPDATE public.user_profiles
   SET role='super_admin', is_app_admin=true,
       group_slug=NULL, group_id=NULL, must_change_password=false
 WHERE email = 'you@nerhadou.com';
```

This is the only account set up by hand. Every other one is created inside the app.

### 3. The user-management function

```bash
npm install -g supabase
supabase login
supabase link --project-ref aqnsrivamxapsjzkajvd
supabase functions deploy admin-users
```

Creating an account needs Supabase's service role key, which bypasses every
security rule in the project. That key cannot go in `index.html` — the file is
public, and anyone who found the key could read or delete everything. So the
function runs on Supabase's servers instead: the browser sends your own login
token, the function checks you really are a super admin, and only then acts.

Supabase provides the key to the function automatically. You never copy it
anywhere, and it must never be committed.

The function only accepts calls from `https://bussiness-excellence.github.io`.
Opening `index.html` straight off your hard drive will fail when you try to add
someone — use the published page.

### 4. Publish the page

Settings → Pages → Source: **Deploy from a branch**, branch `main`,
folder `/ (root)`.

Live at **https://bussiness-excellence.github.io/NERHADOU-CALENDAR/** within a
minute or two. The path is case-sensitive.

### 5. Add the team

Sign in, open the menu under your initials → **Manage people** → **Add person**.

Create the four group managers first, then their teams. Each person gets a
temporary password shown once on screen — hand it over directly. They'll be
asked to choose their own the first time they sign in.

---

## Naming the groups

The groups ship as Group 1 to Group 4. Rename them in the database:

```sql
UPDATE public.groups SET name='Chelation' WHERE slug='g1';
```

Change the names as often as you like. **Never change the slugs** (`g1`–`g4`) —
every event and every person points at them.

---

## How access works

| | Sees | Approves |
|---|---|---|
| Super admin | everything | anything |
| Group manager | everything in their own group | their group's events |
| Marketer | their own events, their group's shared events, team-wide events | nothing |

When someone saves an event they choose who it's for:

- **Just me** — them, whoever it's assigned to, and their manager
- **My group** — everyone in their group
- **Everyone** — the whole marketing team

A marketer's event waits for their manager to approve it before anyone else
sees it. Managers' and admins' own events go straight on the calendar.

These rules live in the database, not in the page. Changing the HTML can hide
buttons, but it can't get at data the rules don't allow.

---

## Day to day

**Someone left.** Manage people → deactivate them. They can't sign in, and
their past events stay on the calendar. Deleting removes their events too.

**Someone forgot their password.** Manage people → the key icon → generate a
new temporary one. They'll be asked to change it when they sign in.

**Changing a manager.** Edit the person's role. A group can only have one
manager, so demote the old one first.

**Don't demote or delete your only super admin.** The app refuses, but it's
worth having a second one so a lost password isn't a crisis.

---

## Notes

- The week starts on Sunday.
- The publishable key in `index.html` is meant to be public. The service role
  key is not, and is never in this repo.
- Every push to `main` goes live straight away. Once people rely on the
  calendar, work on a branch and merge when you're happy.
