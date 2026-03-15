# Keep‑Alive Monitor

This repository automatically prevents your free‑tier projects from being paused due to inactivity. It uses **GitHub Actions** to ping your projects daily and displays the status on a public **GitHub Pages** dashboard.

---

## 🚀 How It Works

- A **GitHub Actions workflow** runs daily at **8:00 AM IST** (2:30 UTC) and can also be triggered manually.
- For each project defined in the workflow matrix:
  1. Calls a PostgreSQL function `ping_keep_alive()` that updates a `keep_alive` table (incrementing `uptime_days` and setting `last_ping` to `now()`).
  2. Fetches the latest `last_ping` and `uptime_days`.
  3. Saves the data as a JSON artifact.
- A separate `combine` job merges all artifacts into a single `docs/data/status.json` file and commits it back to the repository.
- **GitHub Pages** serves `docs/index.html`, which reads `status.json` and displays a clean dashboard showing each project’s last ping time and total uptime.

---

## 📁 Repository Structure

---

## 🛠️ Setup Instructions

### 1. Supabase Database Setup (per project)

In **each** Supabase project, run the following SQL (e.g., in the SQL editor):

```sql
-- Create keep_alive table (single row)
CREATE TABLE public.keep_alive (
  id integer PRIMARY KEY DEFAULT 1,
  created_at timestamptz DEFAULT now(),
  last_ping timestamptz DEFAULT now(),
  uptime_days integer DEFAULT 0,
  CONSTRAINT single_row CHECK (id = 1)
);
INSERT INTO public.keep_alive (id, created_at, last_ping, uptime_days)
VALUES (1, now(), now(), 0)
ON CONFLICT (id) DO NOTHING;

-- Create RPC function to update the table
CREATE OR REPLACE FUNCTION public.ping_keep_alive()
RETURNS void
LANGUAGE sql
SECURITY DEFINER AS $$
  UPDATE public.keep_alive
  SET last_ping = now(),
      uptime_days = uptime_days + 1
  WHERE id = 1;
$$;

-- Grant execute to anon role (used by the workflow)
GRANT EXECUTE ON FUNCTION public.ping_keep_alive() TO anon;
