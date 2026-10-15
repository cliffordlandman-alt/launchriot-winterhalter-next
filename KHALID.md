# Phase 2 backend needs (for Khalid IT): Winterhalter Executive (gareth-api)

Phase 1 preview is browser-only and read-only. Quick capture is stored in the browser (localStorage) only. These are the endpoints needed to make it reliable across Gareth's devices. All per-user (Gareth), cookie-session auth, JSON, same CORS as today. Nothing here sends mail.

Origin to allow for the preview: https://cliffordlandman-alt.github.io (already allowed).

## 1. Decision register
- `GET /decisions?status=open|decided` -> `{items:[{id,title,context,owner,due,status,decided_at,outcome,source_message_id|null,created_at}]}`
- `POST /decisions` {title,context?,owner?,due?,source_message_id?}
- `PUT /decisions/:id` (edit, set status/outcome), `DELETE /decisions/:id`

## 2. Staff task updates (accountability)
Today `/staff-tasks` is derived from CC'd mail and is read-only. Needed:
- `PUT /staff-tasks/:id` {owner,due,priority:low|normal|high,status:open|blocked|awaiting_approval|done,follow_up_at,note}
- `POST /staff-tasks` (manual task not tied to an email) with same fields
- `GET /staff-tasks` to return those fields, plus `?status=awaiting_approval` so "Needs decision" can count real approvals
- Optional: `GET /staff-tasks/summary?from&to` -> per-staff counts opened/closed/overdue

## 3. VIP relationships + notes
- `GET/POST /vips` {name,email|domain,org,kind:client|supplier|colleague,note}, `PUT/DELETE /vips/:id`
- `GET /vips/:id/history` -> recent threads with that address, open commitments
- `POST /vips/:id/notes` {text}, `GET` list, `DELETE` note
- Inbox items should carry `vip: true|false` (match on VIP list) so they rank above ordinary mail.

## 4. Executive memory
- `GET/POST /memory` {kind:style|relationship|standing_instruction|decision_ref,title,body,tags[]}, `PUT/DELETE /memory/:id`
- `GET /memory/search?q=` (plain text search)
- Manually maintained only. Must be exportable and deletable (`GET /export`, `DELETE /memory`).

## 5. Quick capture sync
- `GET/POST /capture` {type:task|note|idea,text,due?,done?}, `PUT/DELETE /capture/:id`. The preview will switch from localStorage to this when it exists.

## 6. Calendar and mail gaps that limit Phase 1
- `/calendar` items need `ends_at`, `attendees[]`, `description` (for meeting prep).
- `/inbox` items: add `replied` reliably, and sent-mail so "waiting for reply" and promises can be tracked.
- Real data: Gareth's Google/Microsoft/Zoho accounts are not connected yet, so all Phase 1 counts are 0 on the live API today.

## 7. Needs decision
Preview defines "Needs decision" as pending `/drafts`. When `awaiting_approval` staff tasks exist, switch the tile to count those too.
