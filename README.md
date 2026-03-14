# Supabase Keep‑Alive Monitor

This repository contains an automated **keep‑alive** system for Supabase projects on the free tier. It prevents projects from being paused due to inactivity by scheduling a daily ping and displays the status on a public GitHub Pages dashboard.

## How It Works

- A **GitHub Actions workflow** runs daily (scheduled) or can be triggered manually.
- For each configured Supabase project, it:
  1. Calls a PostgreSQL function `ping_keep_alive()` that updates a `keep_alive` table (incrementing `uptime_days` and setting `last_ping` to `now()`).
  2. Fetches the latest `last_ping` and `uptime_days` from the table.
  3. Saves all projects’ statuses into a single `docs/data/status.json` file.
  4. Commits and pushes the updated JSON back to the repository.

- A **GitHub Pages** site serves `docs/index.html`, which reads `status.json` and displays a clean dashboard showing each project’s last ping time and uptime.

## Setup

### 1. Create the Supabase Table and Function

In each Supabase project, run the following SQL (e.g., in the SQL editor):

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

-- Grant execute to anon role
GRANT EXECUTE ON FUNCTION public.ping_keep_alive() TO anon;
