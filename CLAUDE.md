# CLAUDE.md — The Lab

Context for Claude Code working in this repo.
Read `PROJECT-BRIEF.md` for the full spec; this is the working agreement.

## What this is
**The Lab** is an ingredient-first flavour pairing R&D tool. Add ingredients,
discover what pairs with them — ranked by confidence. Food and drinks agnostic:
ingredients go in, compatible flavours come out. Output type (dish vs cocktail)
is the user's concern.

Balance, build and batch are cocktail concepts. They live in **The Bar**
(`cocktail-app.html` in the parent folder).

## Stack & conventions
- **One file: `index.html`** — HTML + CSS + vanilla JS. No framework, no build step.
- **Styling:** `tokens.css` (shared with The Bar). CSS variables only. Dark by default.
- **JS style:** `$()` selector helper, `esc()` HTML-escaper, small `render*()` functions,
  plain event listeners. Keep it readable for a non-developer owner.
- Supabase config at the top of the script — placeholders only, owner pastes keys.

## Backend — shared Supabase project (do NOT recreate)
The FlavorGraph engine is already deployed. Call it; never rebuild the tables or RPCs.

```js
const SUPABASE_URL      = "";   // shared with The Bar
const SUPABASE_ANON_KEY = "";   // anon key — safe in client (RLS is read-only)
```

Headers for every request: `{ apikey: KEY, Authorization: "Bearer " + KEY }`.

### Key RPC: `flavor_pair`
```js
POST /rest/v1/rpc/flavor_pair
body: { p_ingredients: ["lamb"], p_mode: "explore", p_exclude: [], p_limit: 20 }
// Returns: [{ name, display_name, role, category, score }]
// score: 0–100, cosine similarity of 300D FlavorGraph embeddings
```
Modes: `explore | garnish | modifier | twist`
→ Classic ↔ Adventurous spectrum in the UI.

### `flavor_tags` RPC
```js
POST /rest/v1/rpc/flavor_tags
body: { p_ingredients: ["lamb", "rosemary"], p_limit: 4 }
// Returns: string[]
```

### Tables
- `GET /rest/v1/flavor_nodes?select=name,display_name,category,bar_role`
  Full ingredient list — use for Ingredient List tab and seed data.
- `recipes` is not needed in The Lab (belongs to The Bar).

**Cache** all responses: memory (`Map`) + `localStorage` under the `"lab:"` prefix.
Degrade gracefully when keys are blank — show no-config banner, disable RPC calls,
fall back to seed data for the Ingredient List.

## FlavorGraph score — what it means
FlavorGraph embeds each ingredient as a 300D vector learned from:
1. Statistical co-occurrence in 1M+ recipes
2. Shared flavor molecule affinities (~1 500 chemical compounds)

The `score` returned by the RPC is cosine similarity of those embeddings, scaled to 0–100.

**Multi-ingredient behavior**: the RPC uses the centroid (mean) of all input embeddings
as the query vector. Only ingredients near that centroid score highly — naturally
producing fewer high-confidence results as more ingredients are added. This is correct
and expected; do not work around it.

## Confidence threshold
Default: **90–100** (well-tested combinations only). A user-facing slider lowers it
to reveal more adventurous options. Always show the threshold control — it's central
to the tool.

## Navigation (three tabs)
`Pairings | Ingredient List | Info`

### Pairings
- Ingredient search/dropdown → builds a list; each item has a padlock toggle and × remove
- 🔒 Locked = fixed in all queries; 🔓 Unlocked = candidate for alternatives
- Results split into two sections (both on this tab):
  - **Pairings**: what else pairs well with the full set — ranked score bars
  - **Alternatives**: for each unlocked ingredient, top swap candidates that raise
    overall compatibility with the locked set
- Confidence threshold slider + Classic ↔ Adventurous toggle
- Results update live on every change

### Ingredient List
- Searchable, filterable table of all `flavor_nodes`
- Filters: category, bar_role
- Row click → flavor profile card (top pairings, category, role); can add to active set

### Info
- Plain-English explanation of FlavorGraph, score derivation, centroid multi-ingredient
  logic, threshold guidance
- Attribution: Park et al., Scientific Reports 2021
- Mirrors content used in tooltips across the rest of the app

## Design principles
- Dense, not padded. Data-forward.
- Tooltips on section headers carry explanations — no inline copy blocks.
- Score bars use `--accent`. A threshold line on the bar shows the cut-off point.
- Locked ingredient: padlock icon + subtle muted border. Unlocked: open padlock.
- All results update live; no submit buttons.

## Don't
- Don't add accounts, a backend, or a toolchain.
- Don't duplicate FlavorGraph tables/RPCs.
- Don't commit real Supabase keys.
- Don't build Balance / ABV / Batch here — those belong in The Bar.
- Don't add cocktail-specific concepts (ml, dilution, ABV) to The Lab.
