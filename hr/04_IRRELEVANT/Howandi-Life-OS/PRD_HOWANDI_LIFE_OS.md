# PRD — HOWANDI LIFE OS (CEO Operating System)

| | |
|---|---|
| **Product** | Howandi Life OS — Personal & Business Operating System |
| **Owner** | Howandi Chandra |
| **Status** | Draft v1.0 |
| **Tanggal** | 5 Juli 2026 |
| **Platform** | Notion (system of record) + Google Calendar + Google Drive + n8n (automation) + AI layer (Claude/GPT) + WhatsApp |

---

## 1. Ringkasan Eksekutif

Howandi menjalankan 10+ "company" sekaligus (Howandi Personal Brand, Laksamana Muda, Laksamana Live, El Oasis, G&G, Global Nusa Niaga, OSF, MC/Public Speaking, Content Creator, Social Activity, Future Investment). Saat ini konteks tersebar di WhatsApp, Notes, Google Calendar, Notion, dan Excel. Akibatnya: kepala penuh, tidak ada satu tempat untuk melihat kondisi hidup dan bisnis secara utuh, dan keputusan diambil tanpa dashboard.

**Yang dibangun bukan project management, tapi Life Operating System (LOS)** — satu sistem yang memperlakukan hidup Howandi sebagai holding company (*Howandi Chandra Inc.*), di mana setiap bisnis adalah anak perusahaan, setiap waktu adalah aset, setiap goal terukur, dan setiap keputusan punya dashboard.

**Target akhir (North Star):** Howandi hanya perlu membuka **satu dashboard setiap pagi** untuk tahu kondisi hidup, bisnis, keuangan, tim, jadwal, dan prioritas hari itu — tanpa berpindah aplikasi.

---

## 2. Latar Belakang & Problem Statement

### 2.1 Masalah inti
Masalahnya bukan terlalu banyak pekerjaan. Masalahnya **tidak ada Operating System** untuk mengorkestrasikan pekerjaan itu. Gejala yang muncul:

- **Fragmentasi konteks** — informasi satu bisnis tersebar di 5 aplikasi berbeda.
- **Tidak ada single source of truth** — task, jadwal, dan catatan hidup di tempat berbeda dan tidak sinkron.
- **Cognitive overload** — otak dipakai sebagai storage, bukan sebagai processor. Kapasitas berpikir strategis tergerus urusan operasional.
- **Blind decision-making** — keputusan (buka cabang, ambil project, hiring) diambil tanpa data terpusat tentang revenue, cashflow, kapasitas, dan prioritas.
- **Tidak ada review loop** — tidak ada mekanisme rutin (harian/mingguan/bulanan) untuk mengukur apakah hidup dan bisnis bergerak sesuai target.

### 2.2 Kenapa sekarang
Skala bisnis sudah melewati titik di mana kepala + WhatsApp + Notes bisa menampung. Tanpa sistem, penambahan bisnis/project ke depan akan memperbesar chaos secara eksponensial, bukan linear.

---

## 3. Tujuan & Metrik Sukses

PRD ini dianggap berhasil bila memenuhi metrik berikut. Setiap metrik punya definisi terukur agar tidak ambigu.

| # | Tujuan | Metrik Sukses | Target |
|---|---|---|---|
| G1 | Satu titik masuk setiap pagi | % hari kerja Howandi membuka **Command Center** sebagai halaman pertama | ≥ 90% dalam 30 hari |
| G2 | Kurangi app-switching | Jumlah aplikasi yang dibuka untuk cek "kondisi hari ini" | Dari 5 → 1 |
| G3 | Semua task punya rumah | % task aktif yang tercatat di sistem (bukan di kepala/WA) | ≥ 95% |
| G4 | Prioritas jelas | Setiap task punya atribut Eisenhower + status eksekusi (4D) | 100% task |
| G5 | Review loop berjalan | Weekly Review + Monthly CEO Review dilakukan tepat waktu | ≥ 90% kepatuhan |
| G6 | Visibilitas bisnis | Setiap bisnis punya snapshot Revenue/KPI yang ter-update | Semua bisnis ter-update ≤ 7 hari |
| G7 | Waktu kembali | Waktu yang dihemat dari cari-cari info & re-planning | Estimasi ≥ 3 jam/minggu (self-reported) |

> **Catatan pengukuran (data discipline):** G7 bersifat *self-reported* → confidence 🟡 sedang. Metrik lain (G1–G6) bisa diukur objektif dari sistem. Jangan campur keduanya saat evaluasi.

---

## 4. Prinsip Desain (Design Principles)

Prinsip ini adalah pagar. Setiap keputusan desain harus lolos prinsip ini.

1. **Single Source of Truth (SSOT).** Setiap jenis data punya SATU rumah kanonik. Waktu → Google Calendar. Semua entity lain (task, project, goal, catatan, dsb.) → Notion. Tidak ada data yang dihitung ganda di dua tempat. Ini non-negotiable — sistem dengan dua sumber kebenaran = sistem yang tidak bisa dipercaya.
2. **One Cockpit.** Halaman default = Command Center. 90% keputusan harian selesai tanpa buka halaman lain.
3. **Everything Has an Owner.** Setiap task, project, dan KPI punya *owner* eksplisit (Do / Delegate / Automate / Delete).
4. **Capture First, Organize Later.** Quick Note / Brain Dump harus 1-tap. Friction menangkap ide = ide hilang.
5. **Progressive Disclosure.** Kompleksitas 21 modul disembunyikan di belakang dashboard. Detail muncul saat di-klik, bukan sekaligus.
6. **Mobile-first untuk capture, Desktop-first untuk review.** Input di mana saja (HP), review mendalam di desktop.
7. **Automate the Boring.** Sinkronisasi, notifikasi, dan ringkasan dikerjakan n8n + AI, bukan manual.
8. **Build in Phases.** Sistem tumbuh bertahap. Jangan bangun 21 modul sekaligus — bangun fondasi dulu, adopsi dulu, baru ekspansi.

---

## 5. Persona & Pengguna

| Persona | Peran | Kebutuhan Utama |
|---|---|---|
| **Howandi (CEO)** | User utama | Overview harian, prioritas, keputusan strategis, review |
| **Assistant / VA** | Operator sistem | Input data, update status, jadwalkan, siapkan meeting notes |
| **Team Lead (per bisnis)** | Kontributor terbatas | Update project & KPI bisnis masing-masing |
| **AI Assistant** | Sistem | Ringkas meeting, susun SOP, generate review, reminder |

**Access model (fase awal):** Howandi = full access. VA = edit terbatas. Team lead = akses ke workspace bisnisnya saja. AI = akses via automation, bukan user manual.

---

## 6. Ruang Lingkup (Scope)

### 6.1 In Scope (keseluruhan produk)
21 modul yang dikelompokkan menjadi 7 domain fungsional (lihat Bagian 8), dibangun bertahap dalam 6 fase (lihat Bagian 12).

### 6.2 Out of Scope (v1)
- Accounting engine penuh (rekonsiliasi otomatis dari rekening bank) — Finance v1 cukup tracking manual + snapshot.
- POS/inventory integration real-time per outlet — masuk backlog.
- Aplikasi mobile native custom — pakai Notion app + WhatsApp bot dulu.
- Multi-user permission granular tingkat enterprise — pakai model akses sederhana dulu.

### 6.3 Definisi MVP (Minimum Viable OS)
MVP = **Fase 1 (Foundation)** saja: **Command Center + Calendar + Task + Project + Goals**. Kalau 5 modul ini dipakai konsisten 30 hari, sistem terbukti *sticky* dan layak diperluas. Kalau tidak dipakai, jangan tambah modul — perbaiki dulu adopsi.

---

## 7. Arsitektur Sistem

**Prinsip: pakai ekosistem, bukan satu aplikasi untuk semua.**

```
                    ┌──────────────────────────┐
                    │   COMMAND CENTER (Notion) │  ← halaman default tiap pagi
                    └────────────┬─────────────┘
                                 │ (rollup & relation)
     ┌───────────────┬───────────┼───────────┬────────────────┐
     ▼               ▼           ▼           ▼                ▼
 Task DB        Project DB    Goals DB   Business DB    Finance/Content/…
 (Notion)       (Notion)      (Notion)   (Notion)       (Notion)
     │               │           │           │
     └───────────────┴─────┬─────┴───────────┘
                           │
              ┌────────────▼────────────┐
              │   n8n (Automation Hub)  │
              └───┬──────────┬─────────┬┘
                  ▼          ▼         ▼
          Google Calendar  WhatsApp   AI Layer (Claude/GPT)
          (SSOT waktu)     (reminder) (ringkas, SOP, review)
                  │
                  ▼
            Google Drive (dokumen)
```

**Peran tiap komponen:**

| Komponen | Peran | SSOT untuk |
|---|---|---|
| **Notion** | Life OS: database, project, knowledge, goals, SOP, jurnal | Semua entity kecuali waktu |
| **Google Calendar** | Jadwal & time blocking | **Waktu** (meeting, deadline, event) |
| **Google Drive** | Penyimpanan file | Dokumen/aset |
| **n8n** | Sinkronisasi task↔kalender, notifikasi, trigger workflow | — (orchestrator) |
| **AI (Claude/GPT)** | Ringkas meeting, buat dokumen/SOP, review mingguan | — (processor) |
| **WhatsApp** | Channel reminder & quick capture | — (interface) |

---

## 8. Modul & Functional Requirements

21 modul dikelompokkan ke **7 domain**. Untuk domain Fase 1 (MVP) diberikan detail penuh (user story + acceptance criteria). Domain fase lanjutan diberikan requirement tingkat fitur — akan diperdalam saat fasenya tiba.

### DOMAIN A — COMMAND CENTER & DASHBOARD `(Modul 1, 21)` — **MVP**

**User story:** *Sebagai CEO, saya ingin satu halaman yang menampilkan seluruh kondisi hari ini, agar saya bisa memutuskan fokus tanpa membuka 20 halaman.*

**Fitur:**
- **Home Dashboard:** Tanggal, Mood, Energi, Today's Priority, Meeting hari ini, Deadline, Top 5 Task, Goal Progress, Financial Snapshot, Habit, Weekly Target, Quick Note/Brain Dump.
- **CEO Command Center:** Today's Focus, Today's Calendar, Critical Task, Waiting Approval, Waiting Others, Overdue, Weekly Goal, Revenue, Cash, Project Progress, Latest Notes, Team Alert, Health, Family Reminder.

**Acceptance Criteria:**
- [ ] Halaman ini terbuka sebagai default view (desktop & mobile).
- [ ] "Top 5 Task" tarik otomatis dari Task DB (filter: Urgent+Important, due hari ini).
- [ ] "Today's Calendar" tampil dari Google Calendar (read).
- [ ] "Overdue" menampilkan task lewat deadline dengan highlight merah.
- [ ] Quick Note bisa diisi dalam ≤ 2 tap dan masuk ke Inbox capture.
- [ ] Semua angka (Revenue, Cash, Goal %) adalah **rollup dari sumber**, bukan input manual di dashboard.

---

### DOMAIN B — TIME & EXECUTION `(Modul 5, 18, 19)` — **MVP (Task, Calendar) + Fase 3 (Habit)**

#### B1. Task Management `Modul 5` — MVP
**User story:** *Sebagai CEO, saya ingin semua task di satu tempat dengan prioritas otomatis, agar saya tahu apa yang harus dikerjakan, oleh siapa, dan kapan.*

**Model prioritas (4D + Eisenhower):**

| Kuadran | Aksi |
|---|---|
| Penting + Mendesak | **Do** — kerjakan hari ini |
| Penting + Tidak Mendesak | **Schedule** — jadwalkan |
| Tidak Penting + Mendesak | **Delegate** — tim |
| Tidak Penting + Tidak Mendesak | **Delete** — hapus |

Status eksekusi tambahan: **Do / Delegate / Automate / Delete.**

**Field wajib per task:** Task, Owner, Business (relation), Deadline, Priority (Urgent flag, Important flag), Estimate, Actual Time, Status.

**Acceptance Criteria:**
- [ ] Setiap task punya atribut Urgent (Y/N) + Important (Y/N) → kuadran terhitung otomatis.
- [ ] Setiap task punya status eksekusi (Do/Delegate/Automate/Delete).
- [ ] Task ter-relasi ke Project dan Business.
- [ ] View tersedia: *Today*, *This Week*, *By Business*, *By Owner*, *Eisenhower Matrix (board)*.
- [ ] Task dengan deadline otomatis muncul di Google Calendar (via n8n).

#### B2. Calendar `Modul 18` — MVP
**User story:** *Sebagai CEO, saya ingin semua jadwal (meeting, deadline, personal) di satu kalender.*

**Acceptance Criteria:**
- [ ] Terhubung ke Google Calendar (SSOT waktu).
- [ ] Tampilan: Today / Week / Month / Quarter / Year.
- [ ] Deadline task & meeting muncul otomatis.
- [ ] Kategori masuk: Meeting, Deadline, Birthday, Holiday, Family Time, Gym, Content Shoot.
- [ ] Perubahan di Calendar **tidak** menciptakan sumber kedua di Notion — Notion menampilkan (mirror), Calendar yang memiliki.

#### B3. Habit Tracker `Modul 19` — Fase 3
Gym, Meditasi, Membaca, Belajar AI, Minum Air, Tidur, Bangun, Stretching, Journal, Prayer — masing-masing dengan **streak counter**.

---

### DOMAIN C — WORK MANAGEMENT `(Modul 4, 11, 15, 16)` — **Project MVP, sisanya Fase 2**

#### C1. Project Center `Modul 4` — MVP
**User story:** *Sebagai CEO, saya ingin semua project lintas bisnis di satu pusat dengan status & progress yang jelas.*

**Field per project:** Owner, Deadline, Priority, Progress (%), Budget, Files (Drive link), Checklist, Meeting, Notes, Timeline, Milestone, Dependencies, Risk, Status.

**Pipeline (Kanban):** Idea → Planning → Research → Ready → Doing → Waiting → Review → Done → Archive.

**Acceptance Criteria:**
- [ ] Board Kanban dengan kolom = pipeline stages.
- [ ] Setiap project ter-relasi ke Business + kumpulan Task.
- [ ] Progress % dihitung dari checklist/task selesai (bukan input manual subjektif).
- [ ] Filter by Business, Owner, Status, Priority.

#### C2. Meeting `Modul 11` — Fase 2
Agenda, Decision, Action Item (→ auto jadi Task), Owner, Deadline, Recording link, **Summary AI**.

#### C3. Team Management `Modul 15` — Fase 2
Per orang: Role, KPI, Meeting, Evaluation, 1-on-1, Warning, Promotion, Training. Grup tim: Laksamana, G&G, GNN, Marketing, Kitchen, Bar, Event, Media, HR, Finance.

#### C4. CRM `Modul 16` — Fase 2
Investor, Supplier, Vendor, Partner, Influencer, Media, Government, Community, Friends, Mentor. Setiap kontak punya **histori interaksi**.

---

### DOMAIN D — BUSINESS OS `(Modul 3, + Life Areas Modul 2)` — **Fase 2**

**User story:** *Sebagai CEO holding, saya ingin membuka satu bisnis dan langsung melihat kesehatannya.*

**Business Database** — entri per bisnis (Laksamana Muda, Laksamana Live, El Oasis, G&G, GNN, OSF, Personal Brand, MC, Social Activity). Saat di-klik menampilkan: Vision, Target, Revenue, KPI, Project, Team, Calendar, Meeting, Task, Documents, SOP, Financial, Asset, Content, Marketing, Notes.

**Life Areas (Modul 2)** — semua project wajib masuk salah satu area: CEO, Family, Health, Spiritual, Relationship, Learning, Business, Finance, Investment, Content, Travel, Social.

**Acceptance Criteria:**
- [ ] Setiap Business = 1 record dengan sub-pages/relations ke Project, Task, KPI, Team.
- [ ] Revenue & KPI ditarik/di-input konsisten (satu definisi per metrik — lihat Bagian 9).
- [ ] Setiap Project & Task wajib punya relation ke Business **dan** Life Area.

---

### DOMAIN E — STRATEGY & GROWTH `(Modul 6, 7, 20, 13, 14)` — **Goals MVP, sisanya Fase 3/5**

#### E1. Goals `Modul 6` — MVP (versi ringan)
Goal 2026: Revenue, Weight, Followers, Books, Gym, Travel, Family, Saving, Investment, Relationship, Reading, Meditation, Sleep. Setiap goal punya **progress bar** (current vs target).

**Acceptance Criteria (MVP):**
- [ ] Minimal 5 goal utama terpasang dengan target angka + progress bar.
- [ ] Progress bar menarik nilai dari sumber bila memungkinkan (mis. Saving dari Finance).

#### E2. Life Roadmap `Modul 7` — Fase 3
Timeline 2026–2030: Rumah, Mobil, Nikah, Anak, HNWI, 10 Cabang, Investor, International.

#### E3. Decision Matrix `Modul 20` — Fase 5
Setiap ide di-score pada: Impact, Revenue, Cost, Time, Difficulty, ROI, Passion, Alignment → output otomatis: High / Medium / Low / Delete.

#### E4. Weekly Review `Modul 13` — Fase 3
Apa berhasil, apa gagal, apa dipelajari, apa dihapus, apa didelegasikan, target minggu depan.

#### E5. Monthly CEO Review `Modul 14` — Fase 3
Score bulanan: Revenue, Profit, Health, Family, Relationship, Business, Content, Learning, Finance, Investment, Leadership, Spiritual.

---

### DOMAIN F — FINANCE & CONTENT `(Modul 8, 9)` — **Fase 3 & 4**

#### F1. Finance OS `Modul 8` — Fase 3
Dashboard: Income, Expense, Cashflow, Investment, Debt, Asset, Liability, Net Worth, Budget, Savings, Insurance. Pisah **Business vs Personal**. Asset register (Rumah, Mobil, Tanah, Perusahaan, Saham, Piutang, Mesin, Furniture, Laptop, Camera) — masing-masing dengan **nilai**.

#### F2. Content OS `Modul 9` — Fase 4
Pipeline: Idea → Script → Shoot → Editing → Schedule → Posted → Analytics → Recycle. Platform: Instagram, TikTok, YouTube, Facebook, Threads, LinkedIn, Website. Content DB per konten: Judul, Hook, CTA, Thumbnail, Status, Topic, Pillar, Business, Target Audience, Views, Leads, Revenue.

---

### DOMAIN G — KNOWLEDGE, AI & PERSONAL `(Modul 10, 17, 12, + Health/Family/Spiritual)` — **Fase 6 & 3**

#### G1. Knowledge Management (Second Brain) `Modul 10` — Fase 6
Kategori: AI, Marketing, Business, Leadership, Finance, Feng Shui, BaZi, Restaurant, Coffee, Prompt, Legal, HR, Management, Book Notes, Meeting Notes, Learning. **Semua searchable.**

#### G2. AI Brain `Modul 17` — Fase 6
Prompt Library, SOP AI, Automation, Workflow. Tools: Claude, ChatGPT, Gemini, Cursor, n8n, MCP.

#### G3. Daily Journal `Modul 12` — Fase 3
Morning (Today's Goal, Affirmation, Priority) + Night (Reflection, Learning, Mistake, Achievement, Mood).

#### G4. Personal (Health, Family, Spiritual) — Fase 3
Masuk sebagai Life Areas + tracker terkait (di Habit & Journal).

---

## 9. Data Model (Ringkasan)

Menerapkan disiplin data: **naming convention konsisten, satu definisi per metrik, relation eksplisit, tanpa data ganda.**

### 9.1 Database inti & relasinya

| Database | Primary key | Relation utama | Catatan SSOT |
|---|---|---|---|
| **Businesses** | Business Name | ← Projects, Tasks, KPIs, Team, Content | Master entity |
| **Projects** | Project Name | → Business (1), → Life Area (1), ← Tasks | Progress = f(task selesai) |
| **Tasks** | Task Name | → Project (0–1), → Business (1), → Owner (1) | Kuadran = f(Urgent, Important) |
| **Goals** | Goal Name | → Life Area (1), → sumber angka (opsional) | Progress ditarik dari sumber |
| **KPIs** | KPI Name | → Business (1), → Owner (1) | Satu definisi per KPI |
| **People** | Name | → Business/Team, ← Tasks (Owner) | Dipakai Team & CRM |
| **Calendar** | (Google Calendar) | mirror ke Notion | **Google Calendar = SSOT waktu** |
| **Notes/Knowledge** | Title | → Topic, → Business (opsional) | Searchable |

### 9.2 Standar KPI (agar angka antar bisnis konsisten)
Setiap KPI didefinisikan dengan format tetap agar tidak ada dua tim menghitung "revenue" secara berbeda:

```
KPI: [nama]
Tipe: leading / lagging
Formula: [definisi matematis eksplisit]
Granularity: harian / mingguan / bulanan
Owner: [siapa yang accountable]
Target: [angka]
Tracking source: [dari mana datanya]
```

> **Wajib:** sebelum KPI dipakai lintas bisnis, definisinya dikunci di satu tempat (KPI dictionary). Ini mencegah "kenapa angka Laksamana beda sama laporan Finance".

### 9.3 Naming convention
- Database: PascalCase plural (`Tasks`, `Projects`).
- Status pakai enum tetap (tidak boleh free-text untuk field yang dipakai filter/rollup).
- Setiap record wajib punya Owner dan minimal satu relation (tidak ada record "yatim").

---

## 10. Automation & Workflows (n8n + AI)

| ID | Workflow | Trigger | Aksi | Fase |
|---|---|---|---|---|
| W1 | Task → Calendar sync | Task dibuat/diubah dengan deadline | Buat/update event di Google Calendar | 1 |
| W2 | Daily briefing | Tiap pagi (jam X) | AI rangkum Command Center → kirim ke WhatsApp | 6 |
| W3 | Overdue alert | Task lewat deadline | Notifikasi WhatsApp ke Owner | 1 |
| W4 | Meeting summary | Selesai meeting / transkrip masuk | AI ringkas → Decision + Action Item jadi Task | 2 |
| W5 | Weekly review draft | Tiap Minggu | AI generate draft Weekly Review dari data minggu itu | 3 |
| W6 | Content scheduler | Konten status "Schedule" | Reminder shoot/post per platform | 4 |
| W7 | Quick capture | Pesan WhatsApp ke bot | Masuk ke Inbox Notion untuk di-triage | 1–2 |

> Beberapa workflow ini bersinggungan langsung dengan project OpenClaw Anda (WhatsApp reminder bot, task bot, broadcast, RAG knowledge bot) — Life OS bisa jadi *consumer* dari bot-bot itu.

---

## 11. Non-Functional Requirements

| Kategori | Requirement |
|---|---|
| **Reliability** | Sinkronisasi Calendar ≤ 5 menit delay. Automation punya error-notification bila gagal. |
| **Performance** | Command Center load ≤ 3 detik di mobile. |
| **Data integrity** | Tidak ada data ganda. Setiap perubahan menghormati SSOT (Bagian 4.1). |
| **Security & Privacy** | Data finansial & personal akses terbatas. Team hanya lihat workspace bisnisnya. API key n8n & AI disimpan aman (env, bukan hardcode). |
| **Backup** | Export/backup berkala (Notion export + Drive). |
| **Usability** | Capture (Quick Note) ≤ 2 tap. Mobile-friendly untuk semua view harian. |
| **Maintainability** | Automation terdokumentasi (apa trigger, apa aksi) agar bisa di-debug. |

---

## 12. Roadmap Bertahap (Phased Rollout)

Prinsip: **jangan bangun 21 modul sekaligus.** Bangun, adopsi, ukur, baru lanjut.

| Fase | Nama | Modul | Deliverable | Exit Criteria |
|---|---|---|---|---|
| **1** | **Foundation (MVP)** | Command Center, Calendar, Task, Project, Goals (ringan) | Cockpit harian + task/project/goal jalan, sync kalender (W1, W3) | Dipakai ≥ 90% hari kerja selama 30 hari |
| **2** | **Business OS** | Business DB, Life Areas, Meeting, Team, CRM | Workspace per bisnis + KPI + tim + meeting (W4, W7) | Semua bisnis punya record + KPI ter-definisi |
| **3** | **Personal OS** | Finance, Journal, Habit, Weekly Review, Monthly CEO Review, Life Roadmap | Modul personal + review loop (W5) | Weekly & Monthly review jalan 4 minggu berturut |
| **4** | **Content OS** | Content OS | Pipeline ide→post→analytics (W6) | 1 bulan konten dikelola penuh di sistem |
| **5** | **Executive Layer** | Decision Matrix + Executive Dashboard | Ringkasan seluruh bisnis/keuangan/target 1 halaman | Keputusan besar lewat Decision Matrix |
| **6** | **Automation & AI** | Knowledge, AI Brain, full automation | Daily briefing WA, AI summary, second brain (W2) | Briefing pagi otomatis + knowledge searchable |

---

## 13. Metrik Adopsi & Definition of Done

**Definition of Done per fase:** exit criteria di Bagian 12 terpenuhi **dan** tidak ada data ganda **dan** Howandi tetap membuka Command Center sebagai default.

**Kill switch:** jika di akhir Fase 1 sistem tidak dipakai konsisten (< 60% hari), **stop menambah modul.** Diagnosa dulu kenapa tidak sticky (terlalu banyak field? capture terlalu ribet? tidak mobile-friendly?) sebelum lanjut. Menambah modul di atas fondasi yang tidak dipakai = memperbesar sistem mati.

---

## 14. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|---|---|---|
| **Over-engineering di awal** — bangun 21 modul sekaligus | Sistem rumit, tidak dipakai | Enforce MVP-first (Fase 1 saja dulu) |
| **Dua sumber kebenaran** (Calendar vs Notion) | Angka/jadwal tidak sinkron | SSOT discipline; Notion mirror, Calendar own |
| **Capture friction** | Ide & task lolos, balik ke WhatsApp | Quick capture ≤ 2 tap + WhatsApp bot (W7) |
| **KPI tidak konsisten antar bisnis** | Dashboard tidak bisa dipercaya | KPI dictionary (Bagian 9.2) dikunci sebelum dipakai |
| **Ketergantungan pada Howandi input manual** | Data basi kalau sibuk | Delegate input ke VA + automation |
| **Maintenance automation** | n8n error tak ketahuan | Error-notification wajib di tiap workflow |

---

## 15. Open Questions / Keputusan yang Harus Diambil

Asumsi yang saya buat di PRD ini — perlu konfirmasi Anda:

1. **Platform system-of-record:** Notion (asumsi, sesuai dokumen) — atau custom-built app? *(confidence 🟡 — dokumen bilang Notion, tapi Anda punya kapasitas build sendiri.)*
2. **Scope MVP:** setuju MVP = 5 modul Fase 1 saja? Atau ada modul lain yang wajib ada di v1?
3. **Siapa operator harian:** Anda sendiri yang input, atau ada VA yang akan meng-input data?
4. **WhatsApp bot:** apakah daily briefing & reminder pakai bot OpenClaw yang sedang Anda bangun, atau tool jadi dulu?
5. **Finance depth:** cukup tracking manual + snapshot, atau perlu integrasi rekening/POS sejak awal?

---

## Appendix A — Peta 21 Modul → Domain → Fase

| # | Modul | Domain | Fase |
|---|---|---|---|
| 1 | Home Dashboard | A | 1 |
| 2 | Life Areas | D | 2 |
| 3 | Business Database | D | 2 |
| 4 | Project Center | C | 1 |
| 5 | Task Management | B | 1 |
| 6 | Goals | E | 1 |
| 7 | Life Roadmap | E | 3 |
| 8 | Finance | F | 3 |
| 9 | Content OS | F | 4 |
| 10 | Knowledge Management | G | 6 |
| 11 | Meeting | C | 2 |
| 12 | Daily Journal | G | 3 |
| 13 | Weekly Review | E | 3 |
| 14 | Monthly CEO Review | E | 3 |
| 15 | Team Management | C | 2 |
| 16 | CRM | C | 2 |
| 17 | AI Brain | G | 6 |
| 18 | Calendar | B | 1 |
| 19 | Habit | B | 3 |
| 20 | Decision Matrix | E | 5 |
| 21 | CEO Command Center | A | 1 |

## Appendix B — Glosarium
- **SSOT (Single Source of Truth):** satu rumah kanonik per jenis data.
- **4D:** Do / Delegate / Automate / Delete.
- **Eisenhower Matrix:** prioritas berdasarkan Urgent × Important.
- **Rollup:** angka yang ditarik otomatis dari database sumber, bukan input ulang.
- **Command Center:** cockpit harian, halaman default.
