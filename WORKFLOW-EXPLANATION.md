# Workflow Explanation — Blade & Crown

Node-by-node documentation of the four n8n workflows shown in the demo video. Node names are transcribed from the n8n canvases visible in the video; on-screen step descriptions are quoted or closely paraphrased. Anything not readable on screen is marked **"Not confirmed from video."**

**Demo facts used throughout:** the barbershop's services include Classic Cut (₹600, 45 min), Skin Fade (₹800, 45 min), and Beard Trim & Shape (₹350, 20 min). Barbers shown: Arjun Mehta (Founder & Master Barber), Marcus Reid (Senior Barber), Kabir Singh (Beard & Shave Specialist), Leo Fernandes (Creative Barber). Website booking steps: Service → Barber → Date & time → Your details → Review. Cancellation policy shown on the booking page: free cancellation/rescheduling up to 2 hours before the appointment; late cancellations/no-shows may be charged 50% of the service price.

---

## Workflow 1 — "Blade & Crown · 1 Retell AI Integration" (Published)

Purpose (on screen): *"Maya's custom tools call this webhook. Every request is authenticated and validated before it touches the database."* This workflow contains **two independent chains**: tool calls and call events.

### Chain A — Tool calls

| # | Node (canvas name) | Type (from icon) |
|---|---|---|
| 1 | Retell Tool Call | Webhook |
| 2 | Authorized Tool Request? | IF |
| 3 | Run Tool in Supabase | HTTP Request (POST) |
| 4 | Return Tool Result / Return Backend Error / Reject Tool Request | Respond-style outputs |

**1. Retell Tool Call**
- **Purpose:** Entry point for every custom tool Maya invokes during a call.
- **Input:** HTTP POST from Retell AI carrying the tool name, tool arguments, and call ID (on-screen wording: *"Webhook receives tool name, args and call ID"*). A canvas sticky note shows the path pattern `POST /webhook/blade-crown-retell-tool?token=…` (full token value never shown; no secret is reproduced here).
- **Processing:** Receives the raw tool-call payload.
- **Output:** Payload passed to the authorization check.
- **Next:** → Authorized Tool Request?

**2. Authorized Tool Request?**
- **Purpose:** Security gate (on screen: *"Secret token + allow-listed tool name"*).
- **Input:** The tool-call payload, including its token.
- **Processing:** Verifies the secret token and that the requested tool name is on the allow-list.
- **Output:** `true` branch → Run Tool in Supabase; `false` branch → Reject Tool Request.
- **Next:** → Run Tool in Supabase (authorized) or → Reject Tool Request (unauthorized)

**3. Run Tool in Supabase**
- **Purpose:** Executes the requested tool inside the database (on screen: *"`agent_tool()` RPC: validation, verification, confirmation"*).
- **Input:** Tool name + arguments from the authorized request.
- **Processing:** HTTP POST to Supabase invoking the `agent_tool()` RPC, which performs input validation, verification, and the confirmation pattern (e.g., `create_appointment` first returns a read-back with nothing booked until `confirmed: true` — visible in the live tool-call panel).
- **Output:** The tool result (e.g., open slots, read-back, or booking confirmation with booking code).
- **Next:** → Return Tool Result (success) or → Return Backend Error (failure)

**4a. Return Tool Result**
- **Purpose:** Returns the result to the voice agent (on screen: *"JSON back to Maya"*).
- **Output:** JSON the Retell agent speaks/uses to continue the conversation. End of chain.

**4b. Return Backend Error**
- **Purpose:** Failure path (on screen: *"…or a safe fallback on error"*).
- **Output:** A safe fallback JSON so the call degrades gracefully instead of breaking. End of chain.

**4c. Reject Tool Request**
- **Purpose:** Terminal node for unauthorized requests (wrong token or non-allow-listed tool). The request never reaches Supabase. End of chain.

### Chain B — Call events

**5. Retell Call Event** (Webhook)
- **Purpose:** Receives Retell call events (e.g., call ended). Sticky note shows the path pattern `POST /webhook/blade-crown-retell-events?token=…`.
- **Next:** → Authorized Event?

**6. Authorized Event?** (IF)
- **Purpose:** Same token authorization pattern as Chain A.
- **Next:** → Save Call to Supabase (authorized); the unauthorized branch's target is **Not confirmed from video**.

**7. Save Call to Supabase** (HTTP Request, POST)
- **Purpose:** Persists the call record (on screen: *"Call metadata & transcript stored for the dashboard"*). The sticky note names the RPC `agent_record_call`.
- **Output:** Call metadata + transcript available to the admin dashboard. End of chain.

### Tools Maya is shown calling through this workflow

| Tool | Evidence |
|---|---|
| `get_available_slots` `{service:"Skin Fade", date:"2026-10-24"}` | Live panel: "ok · 37 open times · 16:00 → Arjun Mehta, Kabir Singh, Leo Fernandes, Marcus Reid" (~1541 ms) |
| `create_appointment` `{…, confirmed: false}` | Live panel: "needs_confirmation · read-back returned, nothing booked yet" (~1825 ms) |
| `create_appointment` `{…, confirmed: true}` | Live panel: "booked ✓ · code 3116E8 · Saturday, October 24 at 4:00 PM" (~859 ms) |

The booking code format and the complete list of allow-listed tool names (e.g., reschedule/cancel tools implied by "book, move or cancel by voice") are **Not confirmed from video** beyond the two tools shown.

---

## Workflow 2 — "Blade & Crown · 2 Appointment & Calendar Sync" (Published)

Purpose (on screen): *"Every booking change writes an outbox row in Supabase, which pushes it here instantly."*

| # | Node (canvas name) | Type (from icon) |
|---|---|---|
| 1 | Calendar Event Queued | Webhook |
| 2 | From Supabase? | IF |
| 3 | Claim Calendar Event | HTTP Request (POST) |
| 4 | Work To Do? | IF |
| 5 | Create, Update or Delete | Switch |
| 6 | Create Google Event / Update Google Event / Delete Google Event | Google Calendar |
| 7 | Mark Synced | HTTP Request (POST) |
| 8 | Record Failure (retry later) | HTTP Request (POST) |

**1. Calendar Event Queued**
- **Purpose:** Entry point for calendar sync jobs (on screen: *"Pushed by Supabase (pg_net) with a shared secret"*).
- **Input:** A queued calendar-event row reference pushed from Supabase the moment a booking changes.
- **Next:** → From Supabase?

**2. From Supabase?**
- **Purpose:** Verifies the push genuinely came from Supabase (shared-secret check).
- **Next:** → Claim Calendar Event (the reject/false path target is **Not confirmed from video**).

**3. Claim Calendar Event**
- **Purpose:** Atomically claims the queued job (on screen: *"Idempotent claim — de-duplicates create vs update"*).
- **Processing:** Marks the outbox row as claimed so duplicate pushes cannot create duplicate calendar events.
- **Next:** → Work To Do?

**4. Work To Do?**
- **Purpose:** Guards against empty/already-processed claims.
- **Next:** → Create, Update or Delete (when there is work); the no-work path is **Not confirmed from video**.

**5. Create, Update or Delete** (Switch)
- **Purpose:** Routes by the claimed action (on screen: *"Switch routes to the right Google Calendar action"*).
- **Next:** → one of the three Google Calendar nodes.

**6. Create / Update / Delete Google Event** (Google Calendar nodes)
- **Purpose:** Writes the change to the salon's Google Calendar (on screen: *"Event written to the salon calendar"*).
- **Output:** On create/update, the Google event ID; on delete, removal confirmation.
- **Next:** → Mark Synced (success) or → Record Failure (error output).

**7. Mark Synced**
- **Purpose:** Writes the result back to Supabase (on screen: *"Event ID stored on the appointment"*; execution view: *"appointment calendar status → synced"*).
- **Next:** End of chain.

**8. Record Failure (retry later)**
- **Purpose:** Persists the failure for retry (on screen: *"Error saved, exponential back-off, retried later"* — retries are re-dispatched by Workflow 4).
- **Next:** End of chain.

### Real execution shown — #82

- Workflow 2 execution **#82**, Oct 21, 13:02:19 — **Succeeded in 1.053 s** (14 KB, tag "Initial calendar sync").
- Scenario: a customer cancellation, end to end in about one second.
- Path: Supabase pushes the change (appointment cancelled) → Claimed: `action = delete` (1 item) → **Delete Google Event** (removed from the calendar) → **Mark Synced** (appointment calendar status → synced).

---

## Workflow 3 — "Blade & Crown · 3 Customer Notifications"

Purpose (on screen): *"Confirmation, reschedule and cancellation emails — plus reminders 24 hours before."* Canvas sticky note: *"Confirmation / reschedule / cancellation rows are pushed here by Supabase. Hourly schedule queues 24h reminders (each one is pushed back here). Failures back off and are retried by workflow 4. Connect your Gmail credential on the Email Customer node."* (The workflow header shows "Publish" rather than "Published" in the video — its live status is **Not confirmed from video**.)

### Chain A — Notification sending

| # | Node (canvas name) | Type (from icon) |
|---|---|---|
| 1 | Notification Queued | Webhook |
| 2 | From Supabase? | IF |
| 3 | Claim Notification | HTTP Request (POST) |
| 4 | Send It? | IF |
| 5 | Email Customer | Gmail |
| 6 | Mark Sent | HTTP Request (POST) |
| 7 | Record Send Failure | HTTP Request (POST) |

**1. Notification Queued** — Webhook receiving notification rows pushed by Supabase (on screen: *"Pushed by Supabase"*). → From Supabase?

**2. From Supabase?** — Origin/shared-secret check, same pattern as Workflow 2. → Claim Notification.

**3. Claim Notification** — Claims the queued notification and prepares the message (on-screen step list: *"Builds subject + HTML email"*). → Send It?

**4. Send It?** — Decision gate before sending (exact condition **Not confirmed from video**). → Email Customer on the send branch.

**5. Email Customer** (Gmail node) — Sends the email to the customer using the Gmail credential connected in n8n. Success output → Mark Sent; error output → Record Send Failure.

**6. Mark Sent** — Records successful delivery back in Supabase. End of chain.

**7. Record Send Failure** — Records the failure; per the step list, *"Retries, then Mark Sent / Record Failure"* — failed sends back off and are retried (Workflow 4 re-dispatches due retries). End of chain.

### Chain B — Reminder queueing

**8. Every Hour** (Schedule Trigger) → **9. Queue Tomorrow's Reminders** (HTTP Request, POST)
- **Purpose:** Every hour, queue reminder notifications for tomorrow's bookings (on screen: *"Tomorrow's bookings get a reminder"*), delivering the "reminders 24 hours before" promise. Queued reminders are pushed back into Chain A for sending.

---

## Workflow 4 — "Blade & Crown · 4 Admin Automation & Monitoring" (Published)

Purpose (on screen): *"A safety net that runs every 30 minutes and reports health to the admin dashboard."* Canvas sticky note: *"Every 30 min: compare upcoming bookings with Google Calendar (re-create missing events), re-dispatch due calendar/email retries, unstick stuck jobs, purge expired voice transcripts, and write a health heartbeat the admin dashboard shows."*

| # | Node (canvas name) | Type (from icon) |
|---|---|---|
| 1 | Every 30 Minutes | Schedule Trigger |
| 2 | Load Calendar Setting | HTTP Request (GET) |
| 3 | List Upcoming Google Events | Google Calendar (getAll: event) |
| 4 | Collect Event IDs | Code |
| 5 | Reconcile, Retry & Purge | HTTP Request (POST) |
| 6 | Retry Without Google Check | HTTP Request (POST) — error branch |
| 7 | Log Google Calendar Error | HTTP Request (POST) — error branch |

**1. Every 30 Minutes** — Schedule trigger; starts the safety-net run. → Load Calendar Setting.

**2. Load Calendar Setting** — HTTP GET loading the calendar configuration/settings from the backend (exact endpoint **Not confirmed from video**; URLs under nodes are truncated in the video). → List Upcoming Google Events.

**3. List Upcoming Google Events** — Reads what the calendar really contains (on screen: *"What the calendar really contains"*). Success output → Collect Event IDs; error output → Retry Without Google Check.

**4. Collect Event IDs** (Code node) — Collects the event IDs from the calendar response so they can be compared with Supabase bookings. → Reconcile, Retry & Purge.

**5. Reconcile, Retry & Purge** — HTTP POST back to the backend to: re-create missing events, retry due calendar/email failures, unstick stuck jobs, purge expired voice transcripts, and write the health heartbeat (on screen: *"Re-create missing events, retry failures, purge old transcripts"*). End of main chain.

**6. Retry Without Google Check** (error branch) — If listing Google events fails, the run still retries due jobs without the calendar comparison. → Log Google Calendar Error.

**7. Log Google Calendar Error** (error branch) — Logs a calendar health warning for the admin dashboard (on screen: *"Still retries, logs a calendar health warning"*). End of chain.

---

## Cross-workflow data flow (summary)

1. **Voice/web action** creates or changes a booking in Supabase.
2. **Supabase pushes** outbox rows via pg_net to Workflows 2 and 3.
3. **Workflow 2** mirrors the change into Google Calendar and marks the appointment synced (or records a failure with back-off).
4. **Workflow 3** emails the customer and marks the notification sent (or records a failure).
5. **Workflow 4** sweeps every 30 minutes: reconciles, retries, purges, and reports health.
6. **Workflow 1** serves Maya's live tool calls throughout, and logs every call for the dashboard.
