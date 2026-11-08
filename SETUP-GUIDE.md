# Setup Guide — Blade & Crown (Reconstruction)

How a developer could recreate the Blade & Crown system. This guide strictly separates:

- ✅ **Verified from video** — visible or stated on screen in the demo.
- 🛠 **Suggested reconstruction** — the author's informed steps to rebuild an equivalent system. These were **not** shown in the video and may differ from the original build.

> 🔐 **No secrets are included anywhere in this guide.** You will create your own tokens and credentials; never copy credentials from screenshots or videos.

---

## 1. Required Software & Services

| Requirement | Verified from video | Notes |
|---|---|---|
| n8n instance with 4 published workflows | ✅ Workflows 1, 2, 4 show "Published"; Workflow 3's status is **Not confirmed from video** | Cloud or self-hosted — hosting choice **Not confirmed from video** |
| Supabase project | ✅ Database, RLS, pg_net pushes, RPC functions `agent_tool()` and `agent_record_call` (names legible on a canvas sticky note) | Postgres + REST/RPC endpoints used by n8n HTTP nodes |
| Retell AI account + voice agent | ✅ Agent "Maya", GPT-4.1, voice "Marissa" | Configured with custom tools + a call-event webhook |
| Google Calendar | ✅ "Salon calendar" written by Workflow 2; read by Workflow 4 | One calendar for the shop |
| Gmail account for sending | ✅ Sticky note: "Connect your Gmail credential on the Email Customer node" | Used by Workflow 3's Gmail node |
| Next.js website | ✅ Labelled on the demo title card | Booking site reading live data from Supabase |

## 2. Credentials You Must Create (never share these)

| Credential | Used by | Verified from video |
|---|---|---|
| Secret token for Retell → n8n webhooks | Workflow 1 authorization IF nodes | ✅ Pattern verified (token value never shown) |
| Shared secret for Supabase → n8n pushes | Workflows 2 & 3 "From Supabase?" checks | ✅ Pattern verified |
| Supabase API credentials for n8n HTTP nodes | Workflows 1–4 | 🛠 Mechanism **Not confirmed from video** |
| Google Calendar credential (OAuth) in n8n | Workflows 2 & 4 | 🛠 Required by the node type; screen never shown |
| Gmail credential in n8n | Workflow 3 "Email Customer" node | ✅ Sticky note instructs connecting it |

## 3. Verified Configuration (from the video)

### Retell agent "Maya"
- ✅ LLM: GPT-4.1; voice: "Marissa" (call-panel header).
- ✅ Capabilities stated on the website banner: "book, move or cancel by voice", 24/7.
- ✅ Custom tools call an n8n webhook with **tool name, args, and call ID**.
- ✅ Tool results return as JSON for Maya to speak; on backend error she receives a **safe fallback**.
- ✅ Call events are sent to a second webhook so metadata + transcript can be stored.
- ✅ Demo tool results: `get_available_slots` returned 37 open times for a service/date; `create_appointment` uses a two-phase confirm (`confirmed:false` → read-back → `confirmed:true`) and returns a booking code (format: 6 characters, e.g. `3116E8` — demo data).

### Supabase
- ✅ Services, prices, barbers, and photos are stored in Supabase and loaded live by the website.
- ✅ Availability is calculated from **schedules, breaks, holidays**, and (per the booking flow) existing bookings — the full caption is cut off in the video; see Limitations.
- ✅ Booking changes write **outbox rows** (calendar events queue; notification queue) that are **pushed to n8n via pg_net with a shared secret**.
- ✅ `agent_tool()` RPC performs "validation, verification, confirmation" for agent tool calls.
- ✅ RLS is enabled ("Supabase + RLS" — end card).

### n8n workflows (names verified exactly)
1. ✅ `Blade & Crown · 1 Retell AI Integration`
2. ✅ `Blade & Crown · 2 Appointment & Calendar Sync`
3. ✅ `Blade & Crown · 3 Customer Notifications`
4. ✅ `Blade & Crown · 4 Admin Automation & Monitoring` — runs **every 30 minutes**

Build each workflow exactly as documented in [`WORKFLOW-EXPLANATION.md`](WORKFLOW-EXPLANATION.md) — every node name there was read from the canvases in the video.

## 4. Suggested Reconstruction Steps 🛠

*(Not shown in the video — a sensible build order for an equivalent system.)*

1. **Supabase first.** Create tables for services, barbers, schedules/breaks/holidays, customers, appointments, a calendar-outbox table, a notification-outbox table, and a calls table. Enable RLS and write policies so the website can read public catalogue data and manage only its own bookings.
2. **Availability function.** Implement a database function that, given service + date (+ optional barber), returns open 15-minute slots after subtracting schedules, breaks, holidays, and existing appointments.
3. **Agent RPCs.** Implement `agent_tool(tool_name, args, call_id)` to dispatch the allow-listed tools (availability, create-with-confirmation, reschedule, cancel, and any others you define) and `agent_record_call(...)` to store call metadata/transcripts.
4. **Outbox pushes.** Use database triggers + pg_net to POST new outbox rows to the Workflow 2 and Workflow 3 webhook URLs, including your shared secret.
5. **Workflow 1.** Webhook → IF (token + allow-list) → HTTP POST to the `agent_tool` RPC → Respond with the JSON result; wire the error output to a fallback response node; build the parallel call-event chain the same way.
6. **Workflow 2.** Webhook → IF (shared secret) → HTTP "claim" call → IF (work to do) → Switch → Google Calendar create/update/delete → HTTP "mark synced"; connect error outputs to an HTTP "record failure" call storing back-off state.
7. **Workflow 3.** Webhook → IF → HTTP "claim notification" (returns recipient, subject, HTML) → IF "Send It?" → Gmail node → HTTP "mark sent" / "record send failure". Add the hourly Schedule chain that queues next-day reminders.
8. **Workflow 4.** Schedule Trigger (30 min) → HTTP "load calendar setting" → Google Calendar "list upcoming events" → Code "collect event IDs" → HTTP "reconcile, retry & purge"; route the calendar node's error output to "retry without Google check" → "log Google Calendar error".
9. **Retell agent.** Create the agent (GPT-4.1, voice of your choice), write a prompt that forces the read-back-and-confirm pattern, register each custom tool against the Workflow 1 webhook URL (with your secret token), and register the call-event webhook.
10. **Website.** Build the Next.js booking flow (Service → Barber → Date & time → Your details → Review) reading catalogue/availability from Supabase, embed the Retell web-call widget, and show the recording/30-day-deletion privacy notice seen in the demo.

## 5. Verification Checklist (mirrors the demo)

- [ ] Ask the voice agent for a slot → availability tool returns real open times
- [ ] Complete a booking by voice → read-back happens **before** anything is booked; booking code is spoken after confirmation
- [ ] The booked slot disappears from the website immediately
- [ ] A Google Calendar event exists for the appointment; the appointment row is marked synced
- [ ] Confirmation email arrives; a reminder is queued for 24 hours before
- [ ] Cancel the appointment → calendar event is deleted (demo execution #82 did this in ~1.05 s)
- [ ] Stop Google Calendar access → Workflow 4 logs a calendar health warning and still retries due jobs
