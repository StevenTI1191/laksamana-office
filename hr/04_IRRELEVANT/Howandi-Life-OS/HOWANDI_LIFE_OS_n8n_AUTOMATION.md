# HOWANDI LIFE OS — n8n Automation Design & Integration Contract

| | |
|---|---|
| **Companion to** | PRD — Howandi Life OS v1.0 |
| **Platform** | Custom-built app + PostgreSQL (SSOT) + n8n (orchestrator) |
| **Status** | Draft v1.0 |
| **Tanggal** | 5 Juli 2026 |

---

## 0. Perubahan Arsitektur (karena pilih Custom-built)

PRD awal mengasumsikan Notion sebagai system-of-record. Dengan pilihan **custom-built app**, arsitektur bergeser:

| Peran | Sebelumnya (Notion) | Sekarang (Custom) |
|---|---|---|
| **SSOT semua entity** | Notion | **PostgreSQL** (DB app Anda) |
| **SSOT waktu** | Google Calendar | Tetap Google Calendar (mirror dari app) |
| **Front-end** | Notion UI | **App custom** (web/mobile) |
| **API** | (Notion API) | **API app Anda sendiri** (wajib dibuat) |
| **n8n** | glue | **Orchestrator** (tidak menyimpan data) |

**Aturan main baru:**
- **PostgreSQL adalah satu-satunya sumber kebenaran.** n8n tidak boleh punya "state" sendiri. Setiap kali n8n butuh data, ia tarik dari API app. Setiap hasil, ia tulis balik ke API app.
- **n8n stateless & idempotent.** Workflow harus aman kalau di-trigger dua kali (tidak bikin event/task ganda).
- **Google Calendar = projection.** App DB adalah master; kalender adalah cermin dari task/meeting yang punya waktu. (Fase awal: sync satu arah app → calendar. Two-way menyusul.)

```
        ┌──────────────────────────────┐
        │   CUSTOM APP (web/mobile)     │  ← UI Howandi
        │   + REST API + Webhooks       │
        └───────┬───────────────┬───────┘
                │ (owns)        │ emit webhooks / expose API
                ▼               ▼
        ┌──────────────┐   ┌─────────────────────────┐
        │ PostgreSQL   │   │        n8n (orchestrator)│
        │  (SSOT)      │   └──┬──────┬──────┬──────┬──┘
        └──────────────┘      ▼      ▼      ▼      ▼
                        Google Cal  WhatsApp  AI   Email
                        (Fonnte/  (Claude/  (Resend/
                         Wablas/   GPT)      Brevo)
                         Meta)
```

---

## 1. Prinsip Integrasi

1. **Contract-first.** n8n dan app berkomunikasi lewat kontrak tetap (API + webhook) di Bagian 3. App Anda wajib mengimplementasikan kontrak ini agar automation jalan.
2. **App owns data, n8n owns orchestration.** Logika bisnis inti (validasi, hitung progress) ada di app. n8n hanya: dengar event → panggil layanan eksternal → tulis balik hasil.
3. **Idempotency wajib.** Setiap workflow yang membuat resource eksternal (calendar event, task) harus cek dulu apakah sudah ada (via ID) sebelum membuat baru.
4. **Auth di semua arah.** App→n8n pakai webhook signing secret. n8n→app pakai API key (bearer token). Semua secret di environment variable, bukan hardcode di node.
5. **Fail loud.** Setiap workflow terhubung ke satu Error Workflow terpusat yang mengirim alert (WA/email) saat gagal. Automation yang gagal diam-diam lebih bahaya daripada tidak ada automation.

---

## 2. Komponen & Environment

### 2.1 Stack
| Komponen | Pilihan | Peran |
|---|---|---|
| Orchestrator | n8n self-hosted (Docker) | Semua workflow |
| Database | PostgreSQL | SSOT (dimiliki app, bukan n8n) |
| WhatsApp | Fonnte / Wablas **atau** Meta Cloud API | Reminder + quick capture |
| AI | Claude (api.anthropic.com) / GPT-4o | Ringkasan, review, briefing |
| Email | Resend / Brevo | Broadcast & notifikasi formal |
| Calendar | Google Calendar API | Projection waktu |
| Hosting | Hetzner / Biznet Gio VPS | n8n + app |

> **Catatan WhatsApp:** untuk workflow *inbound* (quick capture, W7) Anda butuh gateway yang mendukung **incoming webhook**. Fonnte, Wablas, dan Meta Cloud API semuanya bisa; Meta Cloud API paling robust untuk two-way tapi paling banyak setup. Keputusan ini di Bagian 8.

### 2.2 Environment variables (n8n)
```
# App integration
APP_API_BASE_URL      = https://api.howandi-los.internal
APP_API_KEY           = <bearer token n8n→app>
WEBHOOK_SIGNING_SECRET = <verifikasi webhook app→n8n>

# WhatsApp
WA_GATEWAY_URL        = <endpoint kirim pesan>
WA_GATEWAY_TOKEN      = <token>
HOWANDI_PHONE         = 628xxxxxxxxxx

# AI
ANTHROPIC_API_KEY     = <key>   # atau OPENAI_API_KEY

# Google
GOOGLE_OAUTH_...      = <via n8n credential store>

# Ops
N8N_ENCRYPTION_KEY    = <untuk credential n8n>
ALERT_CHANNEL_PHONE   = 628xxxxxxxxxx   # penerima error alert
```

---

## 3. Integration Contract (yang WAJIB app Anda sediakan)

Ini bagian paling penting. Kalau app tidak menyediakan ini, n8n tidak bisa jalan. Rancang API app Anda dengan endpoint & webhook berikut.

### 3.1 REST API — endpoint yang n8n panggil

| Method | Endpoint | Dipakai workflow | Return |
|---|---|---|---|
| `GET` | `/api/tasks/overdue` | W3 | list task lewat deadline + `owner_phone` |
| `GET` | `/api/dashboard/summary?date=today` | W2 | Top 5 task, meeting hari ini, overdue count, goal progress, cash snapshot |
| `POST` | `/api/tasks` | W4, W7 | buat task → return `task_id` |
| `PATCH` | `/api/tasks/:id` | W1 | update task (mis. simpan `calendar_event_id`) |
| `POST` | `/api/inbox` | W7 | simpan quick capture → return `id` |
| `POST` | `/api/meetings/:id/summary` | W4 | simpan summary + decisions |
| `GET` | `/api/reviews/weekly-data?week=YYYY-WW` | W5 | task selesai/gagal, delta goal, revenue minggu itu |
| `POST` | `/api/reviews` | W5 | simpan draft weekly review |
| `GET` | `/api/content/due?date=today` | W6 | konten yang harus shoot/post hari ini |

**Auth:** header `Authorization: Bearer {APP_API_KEY}`.
**Response:** JSON konsisten, selalu sertakan ID resource agar n8n bisa idempotent.

### 3.2 Webhooks — event yang app EMIT ke n8n

App mengirim `POST` ke URL webhook n8n saat event terjadi. Sertakan header signature (HMAC dengan `WEBHOOK_SIGNING_SECRET`) agar n8n bisa verifikasi.

| Event | Payload inti | Memicu workflow |
|---|---|---|
| `task.created` | `task_id, title, deadline, owner, business_id, calendar_event_id(null)` | W1 (create event) |
| `task.updated` | `task_id, deadline, calendar_event_id` | W1 (update event) |
| `task.deleted` | `task_id, calendar_event_id` | W1 (delete event) |
| `meeting.transcript_ready` | `meeting_id, transcript_text` atau `drive_url` | W4 (AI summary) |
| `content.scheduled` | `content_id, platform, publish_at` | W6 (reminder) |

**Contoh payload `task.created`:**
```json
{
  "event": "task.created",
  "task_id": "tsk_01H...",
  "title": "Kirim investor deck G&G",
  "deadline": "2026-07-08T17:00:00+07:00",
  "owner": "Howandi",
  "owner_phone": "628xxxx",
  "business_id": "biz_gng",
  "calendar_event_id": null
}
```

> Kalau app belum siap emit webhook, ada **fallback**: n8n polling `GET /api/tasks?updated_since=...` tiap X menit. Lebih boros tapi lebih cepat dibangun. Webhook adalah target akhir.

---

## 4. Workflow Catalog

| ID | Workflow | Trigger | Layanan | Fase | Prioritas build |
|---|---|---|---|---|---|
| **W1** | Task → Google Calendar Sync | Webhook `task.*` | Google Calendar, App API | 1 | 🔴 pertama |
| **W3** | Overdue Task Alert | Schedule (cron) | App API, WhatsApp | 1 | 🔴 pertama |
| **W7** | WhatsApp Quick Capture | Webhook (WA inbound) | WhatsApp, App API | 1–2 | 🔴 pertama |
| **W2** | Daily Briefing | Schedule 06:00 | App API, AI, WhatsApp | 2 | 🟡 |
| **W4** | Meeting Summary → Action Items | Webhook `meeting.transcript_ready` | AI, App API | 2 | 🟡 |
| **W5** | Weekly Review Draft | Schedule (Minggu malam) | App API, AI | 3 | 🟢 |
| **W6** | Content Scheduler Reminder | Webhook `content.scheduled` / cron | App API, WhatsApp | 4 | 🟢 |
| **W8** | Broadcast (Email/WA terjadwal) | Schedule / manual | Email, WhatsApp | 4 | 🟢 |
| **ERR** | Central Error Handler | Error Trigger | WhatsApp/Email | 1 | 🔴 pertama |

---

## 5. Detailed Workflow Design (Fase 1)

Format node ditulis berurutan. Nama node dalam kurung menunjukkan tipe node n8n.

### W1 — Task → Google Calendar Sync
**Tujuan:** setiap task berdeadline otomatis muncul/terupdate/terhapus di Google Calendar, dan app menyimpan `calendar_event_id`.

```
1. (Webhook) Terima payload task.* dari app
2. (Code) Verifikasi HMAC signature — invalid → stop 401
3. (IF) deadline ada? tidak → stop
4. (Switch) event: created | updated | deleted
   ├─ created:
   │   5a. (Google Calendar → Create) buat event {summary=title, start=deadline}
   │   6a. (HTTP Request) PATCH /api/tasks/:id { calendar_event_id }
   ├─ updated:
   │   5b. (IF) calendar_event_id ada?
   │        ada → (Google Calendar → Update)
   │        tidak → fallback ke cabang created
   └─ deleted:
       5c. (Google Calendar → Delete) pakai calendar_event_id
```
**Idempotency:** cabang `created` hanya membuat event bila `calendar_event_id` masih null. Ini mencegah event ganda kalau webhook terkirim dua kali.
**Error:** semua cabang terhubung ke ERR workflow.

### W3 — Overdue Task Alert
**Tujuan:** tiap pagi (dan/atau tiap jam kerja) kirim WhatsApp ke owner task yang lewat deadline.

```
1. (Schedule Trigger) cron: 08:00 tiap hari (opsional: tiap jam 09–18)
2. (HTTP Request) GET /api/tasks/overdue
3. (IF) hasil kosong? ya → stop
4. (Split In Batches / Loop) per task
5. (Set) susun teks: "⚠️ Overdue: {title} (due {deadline}) — {business}"
6. (HTTP Request) POST WA gateway → kirim ke {owner_phone}
7. (Wait 1–2 detik) hormati rate limit gateway
```
**Idempotency:** aman — hanya kirim notifikasi, tidak mutasi data. (Opsional: tandai `last_alerted_at` via PATCH agar tidak spam tiap jam.)

### W7 — WhatsApp Quick Capture
**Tujuan:** Howandi kirim WA → langsung jadi task/note di app (capture ≤ friction nol).

```
1. (Webhook) terima inbound message dari WA gateway {from, text}
2. (IF) from == HOWANDI_PHONE? tidak → stop (abaikan orang lain)
3. (Code) parse prefix sederhana:
      "t: ..."  → task
      "n: ..."  → note/inbox
      default   → inbox
4. (Switch) task | inbox
   ├─ task:  (HTTP) POST /api/tasks { title, source:"wa" }
   └─ inbox: (HTTP) POST /api/inbox { text, source:"wa" }
5. (HTTP Request) balas WA: "✓ Tercatat: {ringkas}"
```
**Catatan:** parser bisa diperkaya nanti (deteksi bisnis, deadline dari teks) — tapi versi pertama cukup prefix sederhana. Jangan over-engineer parser di awal.

### ERR — Central Error Handler
**Tujuan:** satu tempat menangani semua kegagalan workflow.
```
1. (Error Trigger) menangkap error dari semua workflow (set sebagai Error Workflow di settings)
2. (Set) susun pesan: workflow name + node + error message + timestamp
3. (HTTP Request) kirim ke ALERT_CHANNEL_PHONE via WA (dan/atau email)
```
Set workflow ini sebagai **Error Workflow default** di setiap workflow lain (Settings → Error Workflow).

---

## 6. AI Workflows (Fase 2–3)

Pola panggilan AI: n8n → **HTTP Request** ke `api.anthropic.com/v1/messages` (atau node AI n8n) → parsing → tulis balik ke app. Minta AI **output JSON ketat** agar mudah di-parse.

### W2 — Daily Briefing
```
1. (Schedule) 06:00
2. (HTTP) GET /api/dashboard/summary?date=today
3. (HTTP) AI: prompt "Rangkum jadi briefing pagi ≤120 kata: fokus, meeting, overdue, 1 nudge." + data
4. (HTTP) kirim hasil ke HOWANDI_PHONE via WA
```

### W4 — Meeting Summary → Action Items
```
1. (Webhook) meeting.transcript_ready
2. (IF drive_url) (HTTP/Drive) fetch transcript
3. (HTTP) AI: "Kembalikan JSON {summary, decisions[], action_items[{title,owner,deadline}]}"
4. (Code) JSON.parse aman (try/catch → kalau gagal, kirim ke ERR)
5. (HTTP) POST /api/meetings/:id/summary { summary, decisions }
6. (Loop) action_items → (HTTP) POST /api/tasks per item
```
**Safety:** selalu bungkus parsing JSON AI dengan validasi. AI kadang menambah teks di luar JSON — strip fence ```` ```json ```` dulu.

### W5 — Weekly Review Draft
```
1. (Schedule) Minggu 19:00
2. (HTTP) GET /api/reviews/weekly-data
3. (HTTP) AI: draft "apa berhasil / gagal / dipelajari / dihapus / delegasi / target minggu depan"
4. (HTTP) POST /api/reviews { draft }  → Howandi tinggal edit, bukan nulis dari nol
```

---

## 7. Konvensi & Ops

**Penamaan workflow:** `P{fase}-W{id} {Nama}` — contoh `P1-W1 Task→Calendar`. Beri **tag** per domain (execution / ai / notification).

**Struktur credential:** semua di n8n credential store (terenkripsi via `N8N_ENCRYPTION_KEY`), tidak pernah di dalam node sebagai plain text.

**Monitoring:**
- Aktifkan execution logging (simpan minimal error executions).
- ERR workflow mengirim alert real-time.
- Cek mingguan: berapa execution gagal, node mana paling sering error.

**Backup n8n:** export workflow (JSON) ke Git/Drive secara berkala. Workflow adalah aset — jangan hanya hidup di dalam instance.

---

## 8. Keputusan yang Harus Diambil

| # | Keputusan | Opsi | Rekomendasi awal |
|---|---|---|---|
| 1 | **WhatsApp gateway** | Fonnte / Wablas / Meta Cloud API | Fonnte/Wablas untuk mulai cepat (inbound webhook simpel). Meta Cloud API kalau butuh volume & keandalan resmi. |
| 2 | **Integrasi app↔n8n** | Webhook + API (bersih) vs n8n polling/akses Postgres langsung (cepat) | Mulai **polling/API** biar app belum perlu emit webhook; migrasi ke webhook setelah stabil. |
| 3 | **Arah sync Calendar** | Satu arah (app→cal) vs dua arah | **Satu arah dulu.** Two-way memunculkan konflik sync — tunda sampai perlu. |
| 4 | **AI provider** | Claude vs GPT-4o | Sesuaikan; JSON-mode & prompt di W4/W5 sama-sama didukung. |
| 5 | **Hosting n8n** | Sama VPS dengan app vs terpisah | Sama VPS dulu (hemat), pisahkan kalau beban naik. |

---

## 9. Build Order (rekomendasi eksekusi)

Bangun dari yang memberi nilai tercepat dengan ketergantungan app paling kecil:

1. **ERR** — pasang dulu, biar workflow lain punya jaring pengaman sejak awal.
2. **W7 (Quick Capture)** — nilai instan; hanya butuh 1 endpoint app (`POST /api/inbox`).
3. **W3 (Overdue Alert)** — butuh `GET /api/tasks/overdue`.
4. **W1 (Task→Calendar)** — butuh webhook/patch task; ini yang bikin kalender "hidup".
5. **W2 (Daily Briefing)** — begitu dashboard summary endpoint siap → ini mewujudkan "satu dashboard tiap pagi" langsung ke WA.
6. **W4, W5 (AI)** — setelah meeting & review data tersedia.
7. **W6, W8 (Content & Broadcast)** — Fase 4.

> Target milestone pertama: **ERR + W7 + W3 jalan** → Anda sudah bisa capture via WA dan ditegur soal task overdue, bahkan sebelum app front-end penuh selesai.
