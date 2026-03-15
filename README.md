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

