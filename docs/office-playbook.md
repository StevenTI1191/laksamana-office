# Laksamana Office Playbook

The single reference for how the office platform (`office.laksamanamuda.id`) is built and the rules it follows. Reread this first in a new session. Deep detail on auth lives in [office-auth.md](office-auth.md).

---

## 1. What the platform is

`deploy/` is the office platform, hosted at `office.laksamanamuda.id`. It is a launcher (`deploy/index.html`) plus independent module apps:

- `ordering/` folder, containing two panels: `kitchen/` and `purchasing/`
- `konten/` (COMS, content operations)
- `reservasi/`

Each module is a self-contained single-file HTML PWA. The launcher is the front door and the identity layer.

**Deploy mechanics:** a GitHub Action (`.github/workflows/deploy.yml`) uploads `./deploy/` to Rumahweb via FTP **only on push to `main`**. Local edits and other branches never touch the live site. Anything a module needs at runtime must live inside `deploy/` (e.g. the logo was copied to `deploy/assets/logo.png` because `asset/` outside `deploy/` is not shipped). `docs/` is intentionally outside `deploy/`, so it stays internal.

**Local testing gotcha:** the reservasi module hangs forever on "Menghubungkan ke Server..." under VS Code Live Server. Cause is Live Server's injected live-reload `<script>` mangling the page parse (a SyntaxError before `boot()` runs, so the Google Sheet fetch never fires). It is NOT a code bug. Test reservasi by opening the file directly (`file://.../deploy/reservasi/index.html`, drag onto a tab) or on the real host. This can affect any of the single-file apps; if a module hangs locally with a console SyntaxError citing a line past `</script>`, suspect Live Server first.

**Admin user management (Office landing):** the Office landing (`deploy/index.html`) has a "Kelola User" panel, visible only to owner/admin (session modules `['*']`), that does Users-tab CRUD against the central Sheet via the Apps Script actions `listUsers`/`saveUser`/`deleteUser`. Security: `lm_session` never stores the PIN, so the admin's PIN is captured in an in-memory `adminCreds` at login (or re-prompted once after a reload) and sent as `callerName`/`callerPin` with every admin action; the Apps Script re-verifies the caller is owner/admin before touching the sheet (this re-check IS the enforcement, matching the "enforcement lives in Apps Script" decision). Roles dropdown is populated from the Roles tab. Kitchen/ordering's own in-app user manager is intentionally left hidden (identity is central now). After editing the Apps Script, redeploy a NEW VERSION or the new actions 404.

**Ordering merge (resolved, same pattern as reservasi):** a `deploy/ordering/index (1).html` held yesterday's kitchen feature work (supervisor's per-user role system + a Sheet-syncing user manager) built on the pre-SSO kitchen. Decision: central Office Sheet is the single identity source, so the merge promoted the feature file to the kitchen panel, kept its non-auth features, and replaced its local PIN login + per-module user-sync with SSO Pattern B (`officeUser()` + `resolveOfficeUser()` resolving Office identity against the local role list so per-user roles survive). Its local Kelola User button is hidden. Feature parity verified by marker counts.

**Merge incident (resolved):** a parallel `index_update.html` (yesterday's feature work, sharing tables, move-table, venue seat map, coordination WhatsApp export) was built without today's SSO layer. Merged by promoting `index_update.html` to `index.html`, then re-applying the 5 SSO hunks (head guard, `readLmSession`/`resolveReservasiUser`/`OFFICE_TO_RESERVASI_ROLE`, `boot()` rewrite, `doLogout()` rewrite) and re-running the em-dash/middle-dot punctuation sweep. Verified by feature-marker parity against the original update file and a manual smoke test. Both the update file and the pre-merge backup were deleted after verification. Lesson: when two people (or two sessions) edit the same module in parallel, the SSO layer is the thin, well-isolated part, worth re-applying as precise hunks onto whichever version has the most feature work, rather than trying to hand-port features into the SSO version.

---

## 2. Writing rules (hard bans)

In everything the user reads (chat, code comments, on-screen copy):

- **No em dash** (`—`). Replace with a comma or a full stop. Not a hyphen, en dash, or semicolon.
- **No middle-dot separator** (` · `). Use a full stop or restructure into separate elements.

In-word hyphens in Indonesian (check-in) are fine. As of this writing, `deploy/` and `docs/` are clean of both characters (verified by full-repo grep).

---

## 3. UI guidelines (avoiding AI-design tells)

From the user's "Avoiding AI-Design Red Flags" PDF. Apply the ones that make sense; the office side is internal, so calm over flashy.

- No generic filler copy ("Selamat datang di portal", "Click here"), no invented stats. Write specific, functional microcopy.
- Real logo, not a placeholder icon. One consistent inline-SVG line-icon style, no mixed filled/outline.
- Avoid over-cheery flourishes (a waving-hand emoji in a greeting reads as AI voice).
- Accessibility basics: descriptive alt (empty only if decorative next to adjacent text), semantic `main`/`nav`/`header`/`footer`, one `h1`, visible focus states, and respect `prefers-reduced-motion` (gate every animation behind it).
- Consistency builds trust: one design system across all modules (below).
- The user wants the platform to feel dynamic: tasteful entrance animations, transitions, hover states, always behind `prefers-reduced-motion`.

---

## 4. Design system

Shared stylesheet: `deploy/assets/laksamana.css`. The landing page is its reference implementation. Modules should converge on it.

- Tokens: warm-neutral paper (`--paper #F7F6F4`) + deep gold (`--gold #A9791F`, `--gold-2 #C8961F`, `--gold-dim #7A560F`), ink text (`--ink #2A2620`), `--line #E7E1D3`, status colors, `--ring` focus, `--ease` curve.
- Fonts: Plus Jakarta Sans (headings) + Inter (body), 14px base. (The generic "16px minimum" rule is intentionally not followed; matching the modules matters more.)
- Shared primitives: `.btn` (+ `-primary`/`-ghost`/`-block`), `.pill`, `.badge-live`/`.badge-soon`, `.field`, `.rise` entrance utility, `lm-rise`/`lm-pulse` keyframes.
- Class names stay unprefixed so a module "just works" when it links the stylesheet.

---

## 5. Auth and authorization architecture

Full detail and setup: [office-auth.md](office-auth.md).

**Model:** centralized authentication, user management, and coarse authorization in Office; decentralized business logic in modules. Flow is `Login -> Office -> Module`, one sign-in.

**Trust model:** client-side. It hides modules and removes per-module logins, but a determined user can edit `localStorage`. Real data enforcement, if ever needed, must live server-side (Apps Script / wherever the data is). The model is shaped so enforcement can slot in later.

**Identity store:** a Google Sheet (`Users` + `Roles` tabs) read by an Apps Script web app. The script verifies `name + pin` server-side and returns only that user's identity and module list, so PINs never reach the browser. The `/exec` URL goes in `OFFICE_WEBAPP_URL` in `deploy/index.html`. Empty URL falls back to a local seed so Office runs before the Sheet exists.

**Session contract (`lm_session`):** the API between Office and every module. `{ v, userId, name, role, modules[], issuedAt, expiry }`. `modules` is the resolved leaf keys; `["*"]` means all.

**Module keys are leaf apps:** `kitchen`, `purchasing`, `konten`, `reservasi`. The Ordering launcher card is a group, shown if the user may open `kitchen` OR `purchasing`. This is how role `kitchen` gets Kitchen but not Purchasing.

**Ownership seam:** Office owns identity + which modules you can open (coarse). Each module maps the Office role to its own internal roles/features (fine). Office vocabulary stays small (a handful of roles); features stay private to modules.

---

## 6. Module integration patterns

Every module gets an SSO head guard (first `<head>` script) that redirects to Office before render if there is no valid, authorized session. Then:

- **Pattern A (kitchen, purchasing):** simple role map. `officeUser()` reads `lm_session`, checks the module key, maps owner/admin to internal `admin` else `full`. No user list needed.
- **Pattern B (konten):** resolve against the module's own crew list by id then name; provision newcomers in-memory with a default role from an Office-role map; the owner refines roles in the module's crew screen.

Logout is a single sign-out everywhere: clear `lm_session` (+ any legacy key) and redirect to Office. Old login screens and seed user lists become dead code, left in place to keep changes low-risk.

---

## 7. Migration status and decisions

- **ordering** done (Pattern A). Role map owner/admin -> `admin`, else `full`. Panel-level keys. In-app "Kelola User" hidden (identity is Office now). The team-picker page (`ordering/index.html`) is a session-gated picker, no PINs shipped, and auto-opens the single panel a user is allowed.
- **konten** done (Pattern B). `OFFICE_TO_KONTEN_ROLES`: owner/admin -> `super_admin`, else `content_planner`. Newcomers provisioned in-memory (not force-saved to the live Sheet); owner sets permanent roles in Kelola Kru. Logout was only reachable via the Profile page; added a direct logout icon to the sidebar footer.
- **reservasi** done (Pattern B). `OFFICE_TO_RESERVASI_ROLE`: owner/admin -> `admin`, manager -> `manager`, host/cashier/marketing pass through, unmapped -> `host`. Newcomers provisioned in-memory into `STATE.master.users`; manager/admin sets permanent roles in Master Data. Its sidebar logout icon was already visible, no change needed there.

All four leaf modules (kitchen, purchasing, konten, reservasi) are SSO-migrated. Identity: user's Sheet has both `Users` and `Roles` tabs correctly configured and confirmed working (was the initial blocker for non-owner roles).

**Trust model confirmed by user:** enforcement of who-can-see-what-data lives entirely wherever each Apps Script/Sheet is (server-side per module), not in `lm_session`/Office. Office's client-side gating is accepted as UX-only, by design, not a gap to close.

**Standardization goal:** every module converges on `laksamana.css` and this playbook, so the platform stops reading as four apps built by different people.

**Open items (post-auth-migration):**
1. Shared CSS rollout to ordering/konten/reservasi (still each on its own stylesheet).
2. Dead code cleanup: old local login screens, PIN pads, and seed/user-management UI in kitchen and purchasing were removed; the "Kelola User" management modals (add/edit/delete user) in all four modules are unreachable (their only entry button is permanently hidden) but were deliberately left in place, they're entangled with still-live code (e.g. order-form PIC dropdowns read the same user list). Revisit as a dedicated, carefully-verified pass.
3. ~~Punctuation cleanup~~ Done. All em dashes and middle dots removed from `deploy/` and `docs/` (forecast.js x2, reservasi's ~66 occurrences across toasts, WhatsApp templates, table placeholders, tooltips, and detail panels).
