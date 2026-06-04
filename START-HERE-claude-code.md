# Building The Lab with Claude Code — start here

Claude Code is a command-line tool where you describe what you want and it writes
and edits the files for you. You don't need to write code yourself — you steer.

This folder contains the two reference files Claude Code needs:
`PROJECT-BRIEF.md` (the spec) and `CLAUDE.md` (the working agreement).

---

## One-time setup
1. Install Node.js (LTS) from nodejs.org if you don't have it.
2. Install Claude Code: `npm install -g @anthropic-ai/claude-code`
3. Have your **Supabase URL** and **anon key** handy (same project as The Bar —
   Supabase dashboard → Project Settings → API).

---

## Starting a session

```
cd "<path to>/Cocktail App/flavour-studio"
claude
```

Then at the Claude Code prompt:
```
/init
```
This makes Claude Code scan the folder and read `CLAUDE.md` + `PROJECT-BRIEF.md`.

---

## Paste this as your first message

```
Read CLAUDE.md and PROJECT-BRIEF.md fully before writing code.

You are building The Lab — an ingredient-first flavour pairing R&D tool.
It is a single index.html (HTML + CSS + vanilla JS, no framework, no build step).
It shares tokens.css and the Supabase backend with The Bar (sister app).

The current index.html has an older "Flavour Studio / Build/Batch" structure
that needs to be replaced. Your first task is:

1. Rename the brand to "The Lab" (header, <title>)
2. Strip the Build and Batch tabs entirely (HTML + JS)
3. Replace the tab bar with: Pairings | Ingredient List | Info (stubs for now)
4. Keep the DB client, seed data, and localStorage cache

Then pause and wait for my review before building any tab content.
```

---

## Adding Supabase keys

Once `index.html` exists, tell Claude Code:
```
Fill SUPABASE_URL and SUPABASE_ANON_KEY at the top of index.html with these:
URL: https://YOUR-PROJECT.supabase.co
KEY: YOUR-ANON-KEY
```

## Running locally
```
python -m http.server 8765
```
Open `localhost:8765` in a browser.

## Iterating
One change per request. Examples:
- "Build the Pairings tab — ingredient search, lock/unlock, threshold slider, results list."
- "Add the Alternatives section below the pairings results."
- "Build the Ingredient List tab with search and category filter."
- "Wire the flavor_pair RPC and test it with lamb + rosemary."

---

## Relationship to The Bar
The Bar (`cocktail-app.html` in the parent folder) handles cocktail recipes,
glassware, techniques, and will receive the Build / Balance / Batch features
that were prototyped here. The Lab is pairing-only.
