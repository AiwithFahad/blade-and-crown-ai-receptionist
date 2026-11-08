# Demo Video

## Video link

> 🎬 **Placeholder:** _Add the published demo video URL here (e.g., YouTube / Loom)._
>
> Source video used for this documentation: `Blade & Crown — end-to-end project demo`, 138 s, 1280×720 (local file, not hosted).

## What the video shows

An end-to-end walkthrough of the Blade & Crown barbershop platform:

1. The live website and its real-time booking flow (services, barbers, live slot grid).
2. A live voice call with **Maya**, the Retell AI receptionist, booking a real appointment — with the actual n8n/Supabase tool calls shown on screen.
3. A tour of all four n8n workflows, including a real, successful execution (a cancellation syncing to Google Calendar in about one second).
4. The admin monitoring "safety net" that reconciles the calendar every 30 minutes.

The video states: *"Caller lines are scripted for the demo; every tool call and result is a real request to the production n8n webhook and database."*

## Key timestamps

| Time | Content |
|---|---|
| 00:00 | Website homepage — banner: "talk to our AI receptionist 24/7 — book, move or cancel by voice" |
| ~00:06–00:22 | Live booking slot grid with Maya's voice widget open mid-call |
| ~00:24 | Title card — stack: Next.js · Supabase · Retell AI · n8n × 4 workflows · Google Calendar |
| ~00:35 | Services loaded live from Supabase (Classic Cut ₹600 · Skin Fade ₹800 · Beard Trim & Shape ₹350) |
| ~00:42 | Barbers: Arjun Mehta, Marcus Reid, Kabir Singh, Leo Fernandes |
| ~00:49–00:58 | "Reserve your chair" booking flow (Service → Barber → Date & time → Your details → Review) |
| ~00:52 | On-screen claim: "Every time shown is real availability, calculated from schedules, breaks, holidays, and …" |
| ~01:03–01:22 | **Live call:** Maya books a Skin Fade — `get_available_slots` and two-phase `create_appointment` tool calls shown live |
| ~01:24–01:35 | **Workflow 1 — Retell AI Integration** canvas + 5 explained steps |
| ~01:38–01:48 | **Workflow 2 — Appointment & Calendar Sync** canvas + 6 explained steps |
| ~01:49–01:55 | **Real execution #82 (SUCCEEDED, ~1.05 s)** — cancellation → Delete Google Event → Mark Synced |
| ~01:56–02:03 | **Workflow 3 — Customer Notifications** canvas (confirmations, changes, 24 h reminders) |
| ~02:04–02:11 | **Workflow 4 — Admin Automation & Monitoring** canvas (30-minute safety net) |
| ~02:13–02:18 | End card: "One booking engine. Web and voice. Fully automated." |

## Screenshots in this repo (extracted from the video)

| File | Source timestamp | Content |
|---|---|---|
| `assets/project-demo-thumbnail.png` | ~00:18 | Booking slot grid with Maya speaking in the voice widget |
| `assets/workflow-overview.png` | ~01:31 | Workflow 1 (Retell AI Integration) canvas with all steps highlighted |
| `assets/execution-result.png` | ~01:52 | Workflow 2 execution #82 — "REAL EXECUTION · SUCCEEDED" |
