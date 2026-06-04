# The Lab — Project Brief

An ingredient-first flavour pairing R&D tool. Pick ingredients, discover what
works with them — ranked by confidence from the FlavorGraph engine. Food and
drinks agnostic. The experience is a calm, dense creative workspace.

**Sister app: The Bar** — cocktail building, balancing, batching, recipes.
Both share one Supabase backend.

---

## 1. Goal & guardrails

**Goal:** add one or more ingredients → instantly see ranked pairing suggestions →
explore alternatives → understand why combinations work.

**Non-goals (v1):** user accounts, saving creations, food vs drink mode distinction,
nutrition data, Flavour Bible integration, cocktail-specific concepts (ml, ABV, dilution).

**Audience:** Michael — R&D at his desk. Dense, precise. Tooltips over lengthy copy.

---

## 2. Stack

- Single `index.html` — HTML + CSS + vanilla JS. No build step, no framework.
- `tokens.css` shared with The Bar (same design tokens).
- Supabase REST; same project as The Bar. Anon key in client (RLS read-only).

---

## 3. The FlavorGraph engine (already deployed — do not recreate)

FlavorGraph is a graph neural network trained on 1M+ recipes and ~1 500 flavor
molecules. Each ingredient is a 300-dimensional embedding vector. Similarity between
ingredients = cosine similarity of those vectors, returned as `score` (0–100).

### Multi-ingredient behavior
When multiple ingredients are selected, the RPC uses the **centroid** (mean) of their
embeddings as the query vector. Only ingredients near that centroid score highly —
producing fewer high-confidence results as the set becomes more specific. This is
the intended behavior: broad for "chicken" alone, focused for
"chicken + chilli + garlic + onion".

### RPC endpoints
```
POST /rest/v1/rpc/flavor_pair
  body: { p_ingredients: [...], p_mode: "explore", p_exclude: [], p_limit: 20 }
  returns: [{ name, display_name, role, category, score }]
  modes: explore | garnish | modifier | twist  (Classic → Adventurous spectrum)

POST /rest/v1/rpc/flavor_tags
  body: { p_ingredients: [...], p_limit: 4 }
  returns: string[]

GET /rest/v1/flavor_nodes?select=name,display_name,category,bar_role
```

Cache all responses in memory + `localStorage` (`"lab:"` prefix).

---

## 4. Features

### 4a. Pairings tab (main view)

**Ingredient input**
- Search / dropdown from `flavor_nodes`. Add any number of ingredients.
- Each ingredient in the list has:
  - **Padlock toggle**: 🔒 Locked (fixed in all queries) / 🔓 Unlocked (swap candidate)
  - **× Remove** button
- Adding an ingredient or toggling a lock immediately re-runs the query.

**Controls**
- **Confidence threshold slider**: default 90–100. Drag left to reveal more options.
  A vertical line on each score bar shows the threshold visually.
- **Classic ↔ Adventurous toggle**: 4-position selector mapping to RPC modes:
  `explore` (Classic) → `garnish` → `modifier` → `twist` (Adventurous)

**Results: Pairings section**
- Ranked list of ingredients that pair well with the full ingredient set.
- Each result: ingredient name, category badge, score bar (filled to score value),
  score number. Clicking adds it to the ingredient list.
- Only results at or above the confidence threshold are shown (with a count of hidden
  results below the threshold so the user knows to lower it if needed).

**Results: Alternatives section**
- Shown below the Pairings list, on the same Pairings tab.
- For each **unlocked** ingredient in the list, show top swap candidates ranked by
  how well they fit alongside the **locked** ingredients.
- Format: "Instead of [unlocked ingredient] → [Swap A] (92) / [Swap B] (89) / [Swap C] (87)"
- If all ingredients are locked, this section is hidden with a tooltip:
  "Unlock an ingredient to see swap suggestions."

### 4b. Ingredient List tab

- Full searchable table of all `flavor_nodes`.
- Filters: category (spirit, citrus, herb, spice, dairy…), bar_role.
- Each row: display_name, category, bar_role.
- Clicking a row opens a flavor profile card:
  - Name, category, bar_role
  - Top 10 pairings (from `flavor_pair` in explore mode)
  - "Add to pairing set" button

### 4c. Info tab

Content (succinct, no padding):

**What is FlavorGraph?**
A graph of 8 000 ingredients and flavor compounds, trained on 1M+ recipes and
1 500 flavor molecules. Each ingredient is mapped to a point in a 300-dimensional
space; ingredients close together taste good together.

**What does the score mean?**
Cosine similarity between ingredient embeddings, scaled 0–100.
90+ = well-tested combination. 70–89 = interesting territory. Below 70 = creative risk.

**Multi-ingredient queries**
Adding more ingredients moves the query to the centre of that ingredient cluster.
Only flavours near the centre of all your chosen ingredients score highly — which is
why results narrow as you add more. This is a feature, not a limitation.

**Threshold**
Drag left to explore. Start at 90+ for reliable pairings; go lower to discover.

Attribution: Park et al., "FlavorGraph: a large-scale food-chemical graph for
generating food representations and recommending food pairings," Scientific Reports 2021.

---

## 5. Design principles

- Dense, not padded. R&D tool — data should fill the space.
- Score bars: `--accent` fill, `--surface-alt` track, threshold line in `--warning`.
- Locked state: padlock icon (🔒), chip border in `--border-s`.
- Tooltips on all section headers (title attribute or lightweight tooltip component).
- No submit buttons — everything updates live on input change.
- Graceful degradation: no-config banner when keys are blank; seed data still populates
  Ingredient List.

---

## 6. Definition of done (v1)

- [ ] Pairings tab returns live results from `flavor_pair` RPC
- [ ] Multi-ingredient query narrows results correctly as ingredients are added
- [ ] Confidence threshold slider filters results in real time with visible threshold line
- [ ] Classic ↔ Adventurous toggle changes RPC mode and refreshes results
- [ ] Lock / unlock toggles work; Alternatives show meaningful swap suggestions for unlocked ingredients
- [ ] Ingredient List shows all `flavor_nodes` with search + category/role filter
- [ ] Flavor profile card shows top pairings; "Add to set" works
- [ ] Info tab explains the methodology accurately (centroid logic, score meaning, threshold)
- [ ] Degrades gracefully when Supabase keys are absent
- [ ] Matches The Bar's design tokens (dark, sage-teal accent, Fraunces/Inter)
