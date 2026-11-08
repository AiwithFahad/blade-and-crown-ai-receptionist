# Project Limitations & Disclosure — Blade & Crown

This repository is a **documentation portfolio** created from a single demonstration video (138 seconds, 1280×720). Please read these limitations before evaluating the project.

## Source-of-truth disclosure

- 📹 **All documentation is based solely on the provided demonstration video.** No source code, database access, admin panels, or live systems were inspected.
- 📦 **The original n8n workflow JSON exports are not available** and are not included in this repository. The workflows are documented from the n8n canvases and execution screens visible in the video; they cannot be imported from this repo.
- 🔗 **No live deployment link is available.** The video shows a running website and n8n instance, but no public URL is provided or verified here.
- 💻 **No source code is included** — not for the Next.js website, the Supabase functions, or the Retell agent prompt.

## What could not be verified from the video

| Item | Status |
|---|---|
| Supabase table schema and the SQL inside `agent_tool()` / `agent_record_call` | Function names legible on a canvas sticky note; code never shown |
| Full list of Maya's allow-listed tools (only `get_available_slots` and `create_appointment` are shown) | Partially confirmed |
| Exact webhook URLs | Path patterns partially legible on a sticky note; domains truncated — deliberately not reconstructed |
| Node parameters (credentials, calendar ID, email templates, back-off intervals) | Not shown |
| Workflow 3's published/active status (header shows "Publish", not "Published") | Not confirmed from video |
| Website authentication provider (a sign-in step is shown) | Not confirmed from video |
| Whether availability also reads Google Calendar, or is computed purely in Supabase | Not confirmed from video |
| Payment collection for the stated 50% late-cancellation/no-show fee | Not shown anywhere in the video |
| The tail of the on-screen availability caption ("…calculated from schedules, breaks, holidays, and —") | Cut off in the video frame |
| Performance figures beyond the demo (tool calls ~0.9–1.8 s; execution #82 ~1.05 s) | Demo-only, single samples |
| Hosting, scaling, and cost details for any component | Not shown |

## Demo-data notice

- Names, contact details, dates, and the booking shown in the video are **demo data** (the caller is literally named "Rohan Demo" with an `example.com` email). Personal contact details from the video are intentionally **not reproduced** in this documentation.
- The video itself states: *"Caller lines are scripted for the demo; every tool call and result is a real request to the production n8n webhook and database."* That claim is the presenter's; it is reported here, not independently verified.
- Prices are in INR (₹) as shown on the website in the video.

## How to verify this project properly

Ask the author for: a live walkthrough, the n8n workflow JSON exports, the Supabase schema (sanitized), and the Retell agent configuration (sanitized). Those artifacts — not this documentation — are the authoritative record of the build.
