# Claude Code — kickoff prompt for The Lab

Run `claude` inside the `flavour-studio/` folder, then `/init`, then paste the
block below as your first message.

---

```
Read CLAUDE.md and PROJECT-BRIEF.md fully before writing any code.

You are building **The Lab** — an ingredient-first flavour pairing R&D tool.
Single index.html (HTML + CSS + vanilla JS). Shares tokens.css and the Supabase
FlavorGraph backend with The Bar (sister cocktail app).

The current index.html is a legacy "Flavour Studio" build with cocktail-specific
tabs (Explore / Build / Batch). Rebuild it for the new purpose, in this order:

**Step 1 — Clean and rename**
- Rename brand to "The Lab" (header, <title>, meta)
- Remove Build and Batch tabs (all HTML + JS for ml table, balance, ABV, templates)
- Replace tab bar with: Pairings | Ingredient List | Info (stubs)
- Keep: DB client, seed nodes, localStorage cache, theme toggle, no-config banner
Pause and show me the result.

**Step 2 — Pairings tab**
- Ingredient search/dropdown (from flavor_nodes or seed); selected items form a list
- Each item: padlock toggle (locked/unlocked) + × remove
- Confidence threshold slider (default 90–100)
- Classic ↔ Adventurous toggle (maps to RPC modes: explore → garnish → modifier → twist)
- Pairings section: ranked results (score bar + number); clicking adds to list
- Alternatives section: for each unlocked ingredient, top swap candidates vs the locked set
- All results update live; no submit button
Pause and show me the result.

**Step 3 — Ingredient List tab**
- Searchable, filterable table of all flavor_nodes
- Filters: category, bar_role
- Row click → flavor profile card (top 10 pairings from RPC, category, role, "Add to set")
Pause.

**Step 4 — Info tab**
- Plain-English explanation: what FlavorGraph is, how scores are derived (cosine
  similarity of 300D embeddings), multi-ingredient centroid logic, threshold guidance
- Paper attribution: Park et al., Scientific Reports 2021
- Content mirrors tooltip text used throughout the app
Pause.

**Step 5 — Wire Supabase + test**
- Confirm flavor_pair RPC works with single ingredient ("lamb")
- Confirm multi-ingredient narrowing works ("chicken" vs "chicken + chilli + garlic + onion")
- Show me a screenshot or console log of the raw API response

Working rules:
- One step at a time; wait for my review after each
- No accounts, no backend, no build toolchain
- No cocktail-specific concepts (ml, ABV, dilution) in The Lab
- Never commit real Supabase keys
```
