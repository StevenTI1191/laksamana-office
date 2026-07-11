# Nautical Glossary (Laksamana theme)

The office platform uses a light maritime vocabulary as its personality layer. It started in the Akademi (training) module and is meant to spread across every module as **copy**, not as visual design. The theme lives in the words, so it works on top of whatever design system a module uses.

**Design direction (confirmed): Hybrid.** Modules use the shared office design system (`assets/laksamana.css`: Plus Jakarta Sans, warm paper, deep gold) for visuals, and carry this nautical vocabulary as wording. The Akademi module's own dark-navy + Fraunces look is not the platform style; when Akademi is folded into the office it converges onto `laksamana.css` while keeping the copy below.

Rules for using it:

- **One canonical term per concept.** Pick from the table below. Do not invent synonyms per module ("anjungan" everywhere for dashboard, never also "geladak" or "pusat kendali").
- **Thin, not everywhere.** Apply it to a handful of anchor spots per module (login, logout, greeting, dashboard name, the back-of-house/floor labels). Do not rename every button and field, and do not sprinkle emoji or puns. Overuse reads as gimmicky and, worse, as AI-voice filler.
- **Indonesian first.** These are Indonesian terms for an Indonesian-speaking team. Keep the English brand taglines as-is where they already exist.
- **Respect the writing bans.** No em dash, no middle-dot separator, in any of this copy. See [office-playbook.md](office-playbook.md) section 2.

---

## Core glossary

| Concept | Canonical term | Literal | Notes |
|---|---|---|---|
| CEO / owner | **Nahkoda** | ship's captain | The single person steering. Use for the owner/CEO only. |
| Manager / supervisor | **Perwira** | ship's officer | Optional. Use only if a module needs to name the manager tier; otherwise plain "manager" is fine. |
| Staff / team member | **Kru** (or **Pelaut Muda**) | crew / young sailor | "Kru" is the everyday word. "Pelaut Muda" is the warmer, aspirational variant for onboarding/welcome copy. |
| The team, collectively | **Awak Kapal** | the ship's crew | Use for "the whole team" / roster contexts. |
| Dashboard / home | **Anjungan** | ship's bridge / control deck | The command center. Use as the label for a module's main dashboard view. |
| Floor / venue / dining area | **Dermaga** | dock / pier | Front of house, where guests are received. |
| Kitchen & bar (back of house) | **Galangan** | shipyard | Back of house: prep, cooking, bar. |
| Log in | **Naik kapal** | board the ship | Use in activity logs and optionally on the login action. |
| Log out | **Turun kapal** | disembark | Pairs with "naik kapal". |
| A project / big undertaking | **Pelayaran** | a voyage | Optional, for event/project framing. |
| Going well / on track | **Berlayar lancar** | sailing smoothly | Good-status phrasing. Its opposite: "menghadapi ombak" (facing waves) for at-risk. |
| Guest | **Tamu** | guest | Keep plain. Do not nautical-ify the guest. |

---

## Where each module hooks in

Anchor points only. Everything else stays plain.

**Office launcher (`deploy/index.html`)**
- Greeting stays time-aware ("Selamat pagi/siang/sore/malam, [nama]") and can add a sailing send-off: `greeting-sub` -> "Pilih anjungan untuk mulai berlayar." (currently "Pilih modul di bawah untuk mulai bekerja.")
- Login button "Masuk" can stay literal; the activity/greeting layer is where the theme shows.
- The module grid can be introduced as the fleet: each module card is an "anjungan".

**Login / logout (every module + office)**
- Activity log entries: "[nama] naik kapal" on login, "turun kapal" on logout. (Akademi already does this.)
- Keep the visible button labels "Masuk" / "Keluar" plain for clarity; put the flavor in logs and greetings, not the primary action button.

**Akademi (training)**
- Reference implementation. Already uses Nahkoda, Galangan, Dermaga, Kru, Anjungan, "naik/turun kapal", "Selamat berlayar".

**konten (COMS)**
- Already uses "Kru" for its crew list. Lean in: "Kelola Kru", crew progress.

**reservasi**
- "Dermaga" for the floor/table area label. Reservation flow can frame a booking as securing a spot at the dermaga.

**ordering (kitchen / purchasing)**
- "Galangan" for back-of-house context. Kitchen panel is the Galangan Dapur; bar is Galangan Bar.

**Human Resource / PeopleOS (planned)**
- "Awak Kapal" for the roster / people list. "Anjungan" for the HR dashboard. People Score framed as how well each Kru is sailing.

---

## Status glossary (optional, for dashboards)

Use sparingly, only where a plain status word would otherwise appear.

| Status | Phrase |
|---|---|
| On track / healthy | Berlayar lancar |
| At risk | Menghadapi ombak |
| Blocked / stalled | Berlabuh (anchored) |
| Done / delivered | Sampai di pelabuhan |

Keep real status chips (colors, plain labels like "Selesai", "Terlambat") for anything operational and unambiguous. The phrases above are for summary/headline copy, not for data fields that must stay precise.
