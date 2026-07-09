# Laksamana Muda — Office Portal (`office.laksamanamuda.id`)

Internal operations portal for Laksamana Muda. One domain, one Netlify site, many
self-contained **branches** (apps). This folder (`deploy/`) **is** the website — its
contents are published as-is.

> **Read the [Golden Rules](#-golden-rules-read-before-editing) before touching anything.**
> The whole portal shares one browser origin, so one careless storage key can silently
> corrupt another team's data. Everything else here explains how to avoid that.

---

## 1. What this is (architecture at a glance)

```
office.laksamanamuda.id            → deploy/index.html   (OFFICE HUB — a launcher menu)
│
├── /ordering/                     → ordering/index.html (chooser: Kitchen / Purchasing)
│     ├── /ordering/kitchen/       → kitchen order + check-in app
│     └── /ordering/purchasing/    → purchasing control panel
│
├── /konten/                       → COMS (content operations)
│
└── /reservasi/                    → reservations / guest booking
```

- **One Netlify site.** Publish directory = `deploy/`. Every branch is just a subfolder.
- **One origin.** `https://office.laksamanamuda.id`. All branches share the same
  `localStorage` / `sessionStorage` / cookies. This is the single most important fact.
- **No backend of our own.** Each app talks directly to its own **Google Apps Script
  Web App(s)**, with data living in Google Sheets. The HTML files are 100% static.
- **No build step.** Each app is a single `index.html` with inline CSS + JS (Tailwind or
  hand-written CSS via CDN). Open it in a browser and it runs.
- **Launcher auth model.** The hub has **no login**. Each branch keeps its **own** PIN
  login and its **own** user list. There is intentionally no single sign-on.

---

## 2. Branch map

| Branch | Path | Purpose | Theme | Login |
|---|---|---|---|---|
| **Hub** | `/` | Menu of all modules | Laksamana Muda gold | none (public launcher) |
| **Ordering** | `/ordering/` | Chooser between Kitchen & Purchasing | indigo/emerald | none (just a chooser) |
| Kitchen | `/ordering/kitchen/` | Order belanja + check-in penerimaan | indigo/slate | PIN, multi-account |
| Purchasing | `/ordering/purchasing/` | Order queue, vendors, WA dispatch | indigo | PIN, multi-account + Sheet sync |
| **Konten** | `/konten/` | Content ops (COMS) | gold | Crew + PIN (default 1234) |
| **Reservasi** | `/reservasi/` | Table/guest reservations | gold | Crew + PIN (default 1234) |

To add a new module, see [§6](#6-how-to-add-a-new-branch).

---

## 3. 🔒 Golden Rules (read before editing)

1. **Every web-storage key MUST be prefixed with the branch folder name.**
   `kitchen_*`, `purchasing_*`, `reservasi_*`. (Konten is the one grandfathered
   exception — it uses the `COMS_*` prefix.) **Never** use a bare key like `orders`,
   `users`, `session`, `products`. Because all branches share one origin, a bare key in
   one app **will overwrite** the same-named key in another. This already bit us:
   Kitchen and Purchasing both shipped with `laksamana_users` / `laksamana_session` —
   they were namespaced to fix it. See the [key registry](#4-web-storage-key-registry).

2. **Branches are self-contained. No cross-branch imports.** An app must never read
   another app's files, keys, or globals. If two branches need to share logic, copy it or
   factor it into a clearly-owned shared file — do not reach across folders at runtime.

3. **Relative links only.** Use `../`, `./kitchen/`, `../../` — never hardcode
   `https://office.laksamanamuda.id/...`. This keeps local preview, staging, and
   production all working, and survives a domain change.

4. **One file per app, self-contained.** All HTML + CSS + JS inline in `index.html`
   (plus any explicit sidecar like `laksamana-forecast.js`). No bundler, no npm.

5. **Don't rename a branch folder** without also updating: the `BRANCHES` registry in
   `deploy/index.html`, every relative link that points at it, **and** its storage-key
   prefix.

---

## 4. Web-storage key registry

Single source of truth. If you add a key, add it here. All are `localStorage` unless noted.

| Branch | Keys |
|---|---|
| Hub (`/`) | *(none — stateless launcher)* |
| Ordering chooser | *(none)* |
| **Kitchen** | `kitchen_users`, `kitchen_session`, `kitchen_form_url`, `kitchen_form_id`, `kitchen_appscript_item_url`, `kitchen_appscript_history_url`, `kitchen_appscript_forecast_url`, `kitchen_entry_ids`, `kitchen_batch_picname`, `kitchen_cached_items`, `kitchen_cached_forecast`, `kitchen_stock_today`, `kitchen_stock_asof` |
| **Purchasing** | `purchasing_users`, `purchasing_session`, `purchasing_cafeName`, `purchasing_webAppUrlOrders`, `purchasing_webAppUrlVendors`, `purchasing_webAppUrlItems`, `purchasing_webAppUrlForecast`, `purchasing_webAppUrl` (legacy), `purchasing_vendors`, `purchasing_products`, `purchasing_templateText`, `purchasing_templateConsolidated`, `purchasing_orders`, `purchasing_archivedOrderIds`, `purchasing_cached_forecast` |
| **Konten** | `COMS_SES`, `COMS_LOGIN_LOCK` |
| **Reservasi** | `reservasi_login_lock` (localStorage); `reservasi_session` (**sessionStorage**) |

Prefixes currently in use — **do not reuse these for a new branch**: `kitchen_`,
`purchasing_`, `COMS_`, `reservasi_`.

---

## 5. Auth & data model per branch

Each branch authenticates independently. There is no shared user list.

- **Kitchen** — PIN keypad. User list seeded locally (`SEED_USERS` in the file),
  stored in `kitchen_users`, 24-hour session. Multi-account.
- **Purchasing** — PIN keypad. Seeded locally, **and** can sync the user list from a
  Google Sheet via `USERS_WEBAPP_URL` (set in the file). Roles: `admin` / `full` / `view`.
- **Konten (COMS)** and **Reservasi** — pick a crew member, enter PIN (default `1234`,
  changeable in-app). 3 wrong tries → 1-minute lockout (`*_login_lock`).

**Backends:** every app calls its own Google Apps Script Web App URL(s); data lives in
Google Sheets. Kitchen and Purchasing carry `EMBEDDED_CONFIG` / hardcoded URLs so devices
auto-sync without manual setup. There is no server we host.

**External CDNs** (must be online for full function): Tailwind Play CDN, Font Awesome,
Google Fonts, and JSZip (Purchasing's "download PWA" feature only).

---

## 6. How to add a new branch

1. Create `deploy/<branch>/index.html` — one self-contained file.
2. **Prefix every `localStorage` / `sessionStorage` key with `<branch>_`.** Pick a prefix
   not already listed in [§4](#4-web-storage-key-registry).
3. Add a back link somewhere visible (e.g. login screen): `<a href="../">← Kembali ke Office</a>`.
4. Register it in the `BRANCHES` array near the top of `deploy/index.html`:
   ```js
   { key: 'namabaru', href: './namabaru/', live: true,
     icon: 'fa-solid fa-...', title: 'Nama Baru', tag: 'Deskripsi singkat',
     desc: 'Penjelasan satu kalimat.' }
   ```
   Set `live: false` to show it as a greyed-out "Segera" (coming soon) placeholder.
5. Use **relative** links only, and don't read any other branch's keys/files.
6. Add its keys to the registry table in [§4](#4-web-storage-key-registry).
7. Test: log in, do a real action, then open a **different** branch and confirm nothing
   broke (proves no key collision).

---

## 7. Deployment (Netlify)

- **One site**, publish/base directory = `deploy/`. (Or drag-drop the `deploy` folder into
  Netlify Drop.)
- Custom domain: `office.laksamanamuda.id`.
- Branches are subfolders — no per-branch site or subdomain needed.
- The old `kitchen.laksamana.id` / `purchase.laksamana.id` subdomains can be retired or set
  to redirect to the new paths.

---

## 8. Gotchas & troubleshooting

- **"All my settings/users disappeared."** `localStorage` is **per origin**. Moving to a
  new domain (or first deploy) starts empty. Google Sheet data is safe — but per-app config
  stored locally (Purchasing's Apps Script URLs & WhatsApp templates, Kitchen's form
  URL/entry IDs) must be re-entered **once** on the new domain.
- **A branch clobbered another branch's data.** Someone used a bare/duplicate storage key.
  Check [§4](#4-web-storage-key-registry); enforce the `<branch>_` prefix. This is the #1
  fatal mistake this portal is designed to prevent.
- **Icons/styles missing.** CDN (Tailwind/Font Awesome/Fonts) couldn't load — check the
  network. Offline shows only what a service worker has cached.
- **Stale content after a deploy.** Kitchen & Purchasing register PWA service workers that
  cache their page. Hard-refresh, or unregister the SW in DevTools → Application.
- **Shared default PINs.** Kitchen and Purchasing seed the same demo PINs. That's harmless
  (separate user stores) but change them for real use.

---

## 9. Repo notes

- `deploy/` — **the deployable site** (source of truth for what's live).
- `Pages/` — legacy originals of the Kitchen/Purchasing dashboards, kept for reference.
- Sidecar files: `ordering/kitchen/laksamana-forecast.js`, `forecast_export.js` (and the
  purchasing equivalents) are the shared forecasting module + data export the apps `fetch`.

_Last structural change: consolidated Kitchen + Purchasing under `/ordering/`, added the
Office hub, and namespaced all storage keys per branch._
