# TaskFlow ✦

A full-stack task manager with real user authentication, PostgreSQL database, and one-click deployment.

**Stack:** React + Vite · Supabase (Auth + PostgreSQL) · Vercel

---

## Features

- ✅ Real email/password authentication (signup, login, logout)
- ✅ PostgreSQL database with row-level security (users only see their own data)
- ✅ Create, complete, and delete tasks
- ✅ Priority levels (Low / Medium / High)
- ✅ Filter tasks (All / Active / Completed)
- ✅ Progress tracking
- ✅ Fully deployed (Supabase backend + Vercel frontend)

---

## Setup Guide

### Step 1 — Create a Supabase Project (free)

1. Go to [supabase.com](https://supabase.com) and sign up / log in
2. Click **"New project"** and fill in the details
3. Wait ~1 minute for it to provision

### Step 2 — Create the Database Table

In your Supabase project, go to **SQL Editor** and run this:

```sql
-- Create the tasks table
CREATE TABLE tasks (
  id          uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id     uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  title       text NOT NULL,
  priority    text NOT NULL DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high')),
  done        boolean NOT NULL DEFAULT false,
  created_at  timestamptz DEFAULT now() NOT NULL
);

-- Enable Row Level Security
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- Policy: users can only access their own tasks
CREATE POLICY "Users manage own tasks"
ON tasks
FOR ALL
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);
```

### Step 3 — Get Your API Keys

In your Supabase project: **Settings → API**

Copy:
- **Project URL** → looks like `https://abcdefgh.supabase.co`
- **anon / public key** → the long JWT string

### Step 4 — Configure Environment Variables

Copy the template and fill in your keys:

```bash
cp .env.example .env
```

Edit `.env`:

```
VITE_SUPABASE_URL=https://your-project-id.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-public-key-here
```

### Step 5 — Run Locally

```bash
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173)

---

## Deployment (Vercel — Free)

### Option A: Deploy via Vercel CLI

```bash
npm install -g vercel
vercel

# Set environment variables when prompted, or add them in Vercel dashboard
```

### Option B: Deploy via GitHub (recommended)

1. Push this repo to GitHub:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR_USERNAME/taskflow.git
   git push -u origin main
   ```

2. Go to [vercel.com](https://vercel.com) → **Add New Project** → Import your GitHub repo

3. In Vercel project settings, add **Environment Variables**:
   - `VITE_SUPABASE_URL` = your Supabase URL
   - `VITE_SUPABASE_ANON_KEY` = your Supabase anon key

4. Click **Deploy** — you'll get a live URL in ~30 seconds!

---

## Optional: Disable Email Confirmation

By default Supabase requires email confirmation. To skip this during development:

**Supabase Dashboard → Authentication → Providers → Email → Disable "Confirm email"**

---

## Project Structure

```
taskflow/
├── src/
│   ├── lib/
│   │   ├── supabase.js        # Supabase client
│   │   └── AuthContext.jsx    # Auth state & helpers
│   ├── pages/
│   │   ├── Login.jsx          # Login page
│   │   ├── Signup.jsx         # Signup page
│   │   ├── Dashboard.jsx      # Main app (tasks)
│   │   ├── auth.css           # Auth page styles
│   │   └── dashboard.css      # Dashboard styles
│   ├── App.jsx                # Router + protected routes
│   ├── main.jsx               # Entry point
│   └── index.css              # Global styles
├── .env.example               # Environment variable template
├── vercel.json                # Vercel SPA routing config
├── vite.config.js
└── package.json
```

---

## Security Notes

- The `anon` key is safe to expose in frontend code — it's designed for this
- Row Level Security (RLS) ensures users can only read/write their own data
- Passwords are handled entirely by Supabase Auth (bcrypt hashed, never stored in plain text)
- JWTs are stored in localStorage by the Supabase client

---

## License

MIT
