# evenzoo

**Event management system built for the Computer Society & Gaming Club, IAR.**

> Live at [evenzoo.netlify.app](https://evenzoo.netlify.app)

evenzoo handles the full lifecycle of a college club event — from creation and registration to QR-based attendance and post-event feedback — in a single HTML file backed by Supabase.

---

## Features

**For Students**
- Sign up with name, email, IAR number, course, and department
- Browse upcoming and past events
- Register for events and receive a unique QR code
- View attendance status
- Submit star ratings and comments after attending

**For Core Members**
- Create, edit, and delete events with optional capacity limits
- Scan student QR codes via camera or manual entry to mark attendance
- Duplicate scan detection — same QR can't be marked twice
- View per-event participant list with registration and attendance status

**For Admins**
- Everything core members can do
- Promote or demote any user between Student, Core, and Admin roles
- View analytics — registrations, attendance rates, average ratings, feedback log

---

## Tech Stack

| Layer | Tool |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (single file) |
| Backend | [Supabase](https://supabase.com) (Postgres + Auth) |
| Hosting | [Netlify](https://netlify.com) |
| QR Generation | [qrcodejs](https://github.com/davidshimjs/qrcodejs) |
| QR Scanning | [BarcodeDetector API](https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector) + [jsQR](https://github.com/cozmo/jsQR) fallback |

---

## Database Schema

```sql
-- Profiles (extends Supabase auth.users)
create table public.profiles (
  id uuid primary key references auth.users(id),
  name text not null,
  role text not null default 'student' check (role in ('student','core','admin')),
  iar_no text,
  course text,
  department text,
  created_at timestamptz default now()
);

-- Events
create table public.events (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  date date not null,
  venue text not null,
  description text,
  capacity integer,
  created_by uuid references public.profiles(id),
  created_at timestamptz default now()
);

-- Registrations
create table public.registrations (
  id uuid primary key default gen_random_uuid(),
  event_id uuid references public.events(id),
  user_id uuid references public.profiles(id),
  created_at timestamptz default now()
);

-- Attendance
create table public.attendance (
  id uuid primary key default gen_random_uuid(),
  event_id uuid references public.events(id),
  user_id uuid references public.profiles(id),
  marked_at timestamptz default now()
);

-- Feedback
create table public.feedback (
  id uuid primary key default gen_random_uuid(),
  event_id uuid references public.events(id),
  user_id uuid references public.profiles(id),
  rating integer not null check (rating between 1 and 5),
  comment text,
  created_at timestamptz default now()
);
```

---

## RLS Policies

Run this in your Supabase SQL Editor to allow authenticated users to access the data:

```sql
-- Schema access
grant usage on schema public to anon, authenticated;
grant select, insert, update, delete on public.events to authenticated;
grant select, insert, update, delete on public.profiles to authenticated;
grant select, insert, update, delete on public.registrations to authenticated;
grant select, insert, update, delete on public.attendance to authenticated;
grant select, insert, update, delete on public.feedback to authenticated;

-- RLS
alter table public.events enable row level security;
alter table public.profiles enable row level security;
alter table public.registrations enable row level security;
alter table public.attendance enable row level security;
alter table public.feedback enable row level security;

create policy "authenticated read/write" on public.events for all using (auth.uid() is not null);
create policy "authenticated read/write" on public.profiles for all using (auth.uid() is not null);
create policy "authenticated read/write" on public.registrations for all using (auth.uid() is not null);
create policy "authenticated read/write" on public.attendance for all using (auth.uid() is not null);
create policy "authenticated read/write" on public.feedback for all using (auth.uid() is not null);
```

---

## Setup

1. **Clone the repo**
```bash
git clone https://github.com/yourusername/evenzoo.git
cd evenzoo
```

2. **Create a Supabase project** at [supabase.com](https://supabase.com), run the schema and RLS SQL above in the SQL Editor.

3. **Plug in your credentials** — open `evenzoo.html` and replace these two lines near the top of the `<script>` tag:
```js
var SURL = 'your-supabase-project-url';
var SKEY = 'your-supabase-anon-key';
```

4. **Serve locally** using VS Code Live Server or:
```bash
python -m http.server 5500
```
Then open `http://127.0.0.1:5500/evenzoo.html`

5. **Deploy** — drag and drop `evenzoo.html` onto [netlify.com/drop](https://netlify.com/drop).

6. **Set your first admin** — sign up normally, then run this in Supabase SQL Editor:
```sql
update public.profiles
set role = 'admin'
where id = (select id from auth.users where email = 'your@email.com');
```

> **Note:** Disable email confirmations in Supabase → Authentication → Settings for a smoother signup experience at events.

---

## Role Access

| Feature | Student | Core | Admin |
|---|:---:|:---:|:---:|
| Register for events + QR | ✓ | — | — |
| Submit feedback | ✓ | — | — |
| Create / Edit / Delete events | — | ✓ | ✓ |
| QR scanner (mark attendance) | — | ✓ | ✓ |
| View participant lists | — | ✓ | ✓ |
| Analytics | — | ✓ | ✓ |
| Promote / Demote users | — | — | ✓ |

---

## Roadmap

- [ ] Real-time core member chat (Supabase channels)
- [ ] Email confirmations and notifications
- [ ] Event image uploads
- [ ] Export attendance and feedback to CSV
- [ ] PWA support for mobile install

---

## Built by

Hetavi Makwana
