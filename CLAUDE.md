# CLAUDE.md — The Lab

Context for Claude Code working in this repo. This is the working agreement.

## What this is
**The Lab** is an ingredient-first flavour pairing R&D tool. Add any ingredients —
food or drinks — and get ranked pairing suggestions from the FlavorGraph engine.
Food and drinks agnostic: the tool doesn't know or care whether you're building
a cocktail or a dish.

**Sister app: The Bar** — cocktail recipe browser, glassware, techniques.
Both share the same Supabase backend.

---

## File layout

```
index.html   ← Landing page (Zenith-X design, links to app.html and The Bar)
app.html     ← The Lab tool  ← THIS IS THE FILE TO EDIT
tokens.css   ← Shared design tokens (sage-teal accent, Fraunces/Inter)
CLAUDE.md    ← This file
PROJECT-BRIEF.md ← Full feature spec
```

**When editing the tool, edit `app.html`. Never edit `index.html` for tool changes.**

The landing page (`index.html`) is identical to The Bar's — keep both in sync when
making landing page changes. The only difference is which door's Enter button links
to `./app.html` vs to the sister Vercel URL.

---

## Stack & conventions
- **Single file: `app.html`** — HTML + CSS + vanilla JS. No framework, no build step.
- **Styling:** `tokens.css`. CSS variables only. Dark by default (`data-theme="dark"`).
- **JS style:** `$()` selector helper, `esc()` HTML-escaper, `const` / arrow functions,
  small render functions, plain event listeners.
- App name in all user-facing text: **The Lab.**

---

## Backend — shared Supabase project (do NOT recreate)

```js
const SUPABASE_URL      = "https://zmflwqfebartfnjrsvpv.supabase.co";
const SUPABASE_ANON_KEY = "sb_publishable_paTGMPndKkktfbuqxpdLCA_pkPmHkYB";
```

**Auth header — critical:** The key is a publishable key, not a JWT.
Send it in `apikey` only. `Authorization: Bearer` returns 401.

```js
// CORRECT
headers: { apikey: SUPABASE_ANON_KEY, "Content-Type": "application/json" }

// WRONG — returns 401
headers: { apikey: SUPABASE_ANON_KEY, Authorization: "Bearer " + SUPABASE_ANON_KEY }
```

The `DB` object in `app.html` already has this correct; don't change it.

---

## FlavorGraph RPCs

```js
// Pairings for a set of ingredients
POST /rest/v1/rpc/flavor_pair
body: { p_ingredients: ["lamb"], p_mode: "explore", p_exclude: [], p_limit: 40 }
// Returns: [{ name, display_name, role, category, score }]
// score: raw cosine similarity, 0–1. The app normalises to 0–100 via normPairings().
// Modes: explore | garnish | modifier | twist  (Classic → Adventurous spectrum)

// Flavour tag labels for a set of ingredients
POST /rest/v1/rpc/flavor_tags
body: { p_ingredients: ["lamb", "rosemary"], p_limit: 4 }
// Returns: string[]

// All ingredient nodes
GET /rest/v1/flavor_nodes?select=name,display_name,category,bar_role
```

**Score normalisation:** The RPC returns raw cosine similarity (0–1). `normPairings()`
in `app.html` multiplies by 100 when `score <= 1.0`. Do not remove this.

**Score reality:** Most real-world pairings score 10–50 out of 100. The confidence
threshold defaults to **0** (show all). Users raise it to filter to higher-confidence
pairings. This is intentional — a default of 90 returns nothing for most ingredients.

**Cache:** All RPC responses cached in memory (`Map`) + localStorage under `"lab:"` prefix.
A one-time cache flush (keyed on `lab:cache-v`) runs on load to clear stale data.

---

## What's built (app.html)

| Tab | Status |
|---|---|
| Pairings | ✅ Complete — multi-ingredient input, lock/unlock, threshold slider, style modes, score bars, Alternatives section |
| Ingredient List | ✅ Complete — searchable table of 6,600+ nodes, role/category filter, profile overlay |
| Info | ✅ Complete — FlavorGraph explanation, centroid logic, score guide, attribution |

Theme toggle (dark/light), graceful offline fallback to seed data — both working.

---

## What's NOT built

- URL routing / deep links (no back-button support; each session starts fresh)
- "Save / export session" — bench is in memory only
- Pairings for non-ingredient concepts (the FlavorGraph covers food + drink nodes only)

---

## Don't
- Don't add accounts or a build toolchain.
- Don't recreate FlavorGraph tables/RPCs — only read them.
- Don't commit real Supabase keys (the current key is already committed — it's a
  read-only publishable key, safe in client code with RLS).
- Don't build Balance/ABV/Batch/Build here — those belong in The Bar.
- Don't add cocktail-specific concepts (ml, dilution, ABV) to The Lab.

---

## Deployments
- **Live:** https://the-lab-pearl-tau.vercel.app (auto-deploys from `main`)
- Landing page: `/` (index.html)
- Tool: `/app.html`
- **Repo:** https://github.com/SaturdayLim/the-lab
