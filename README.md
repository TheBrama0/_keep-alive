# Keep‑Alive Monitor

This repository automatically prevents your free‑tier **Supabase** projects and **Render** backend from being paused due to inactivity. It uses **GitHub Actions** to ping your services regularly and displays the status on a public **GitHub Pages** dashboard.

---

## 🚀 How It Works

### Supabase Projects (Database keep‑alive)
- A GitHub Actions workflow (`.github/workflows/keep-alive.yml`) runs daily at **8:00 AM IST** (2:30 UTC) and can also be triggered manually.
- For each Supabase project defined in the workflow matrix:
  1. Calls a PostgreSQL function `ping_keep_alive()` that updates a `keep_alive` table (incrementing `uptime_days` and setting `last_ping` to `now()`).
  2. Fetches the latest `last_ping` and `uptime_days`.
  3. Saves the data as a JSON artifact.
- A `combine` job merges all Supabase artifacts into `docs/data/status.json` and commits it.

### Render Backend (Web service keep‑alive)
- Another workflow (`.github/workflows/ping-render.yml`) runs **every 10 minutes** to keep the Render web service active.
- It pings the health endpoint `https://track2link.onrender.com/status/keepalive` and captures:
  - HTTP status code
  - Timestamp
- The result is saved as `docs/data/render-status.json` and committed.

### Unified Dashboard (GitHub Pages)
- `docs/index.html` loads **both** `status.json` (Supabase) and `render-status.json` (Render).
- Displays:
  - **Supabase projects**: last ping time, uptime days
  - **Render backend**: last ping time, HTTP status (alive/unreachable)
- The page auto‑refreshes every 30 seconds to show near‑real‑time status.

---

## 📁 Repository Structure
