Here's a detailed technical plan for the **PokéWordle** game:

---

## 🗂️ Architecture Overview

**Single-page HTML/CSS/JS app** — no build tools needed, runs entirely in the browser.

---

## 📡 Data Sources

| Source | What it's used for |
|---|---|
| `pokeapi.co/api/v2/pokemon/{id}` | Types, base stats, sprites, name, game indices |
| `pokeapi.co/api/v2/pokemon-species/{id}` | Generation, evolution chain URL, region (habitat/version groups), flavor text |
| `pokeapi.co/api/v2/evolution-chain/{id}` | Walking the chain to determine evolutionary stage |
| PokeAPI Sprites | Official front sprite for silhouette rendering via CSS |
| Bulbapedia (static lookup table) | Pokémon region of origin — PokeAPI's `generation` maps to region, but a hardcoded lookup table (Gen → Region) is the most reliable and avoids extra fetches |

> **Note on Bulbapedia:** Bulbapedia doesn't have a public API, so the plan is to use a **hardcoded JSON lookup table** derived from Bulbapedia's data (e.g. Gen I → Kanto, Gen II → Johto, etc.) rather than scraping it live. This is standard practice for PokéAPI-based projects.

---

## 🧠 Game State & Logic

### On Page Load
1. Pick a **random Pokémon ID** (1–1025, skipping forms)
2. Fetch from `pokemon/{id}` and `pokemon-species/{id}` in parallel
3. Fetch `evolution-chain/{id}` from the species data
4. Compute and cache all 6 hint values:

```
{
  nameLength: 7,
  generation: "Generation II",
  evoStage: "Stage 2 (Middle)",   // position in evo chain
  region: "Johto",
  types: ["Fire", "Flying"],
  spriteUrl: "https://raw.githubusercontent.com/.../charizard.png"
}
```

### Evolutionary Stage Logic
Walk the evolution chain tree recursively:
- **No evolutions** → "Standalone (Does not evolve)"
- **First in chain** → "Basic (First Stage)"
- **Middle** → "Stage 1 (Middle Evolution)"
- **Final** → "Stage 2 / Final Evolution"
- **Baby Pokémon** → detected via `is_baby` flag on species

---

## 🎮 The 6 Steps

| Step | Trigger | Reveal |
|---|---|---|
| **1** | Page load | "The Pokémon's name has **7 letters**" |
| **2** | After 1st wrong guess | "This Pokémon is from **Generation II**" |
| **3** | After 2nd wrong guess | "This is a **Final Evolution**" |
| **4** | After 3rd wrong guess | "This Pokémon originates from **Johto**" |
| **5** | After 4th wrong guess | "This Pokémon's type is **Fire / Flying**" |
| **6** | After 5th wrong guess | Show silhouette (black CSS filter on sprite) |

---

## ✏️ Input & Guessing

- Text input with **autocomplete dropdown** sourced from a full local Pokémon name list (fetched once from `pokeapi.co/api/v2/pokemon?limit=1025`)
- On submit: **case-insensitive string match** against the answer
- **Correct** → Win screen with full sprite reveal + Pokédex flavor text
- **Wrong** → Increment step counter, reveal next hint, shake the input
- **6 wrong guesses** → Game over screen, reveal the Pokémon

---

## 🎨 UI Components

```
[ Header: "PokéWordle" ]
[ Hint Cards — revealed one by one, stacked vertically ]
[ Guess Input + Autocomplete Dropdown ]
[ Previous Guesses list (with ❌ marker) ]
[ Win/Lose Modal with full reveal ]
```

**Silhouette effect:** render the sprite in an `<img>` tag with CSS:
```css
filter: brightness(0);  /* pure black silhouette */
```
On win, animate `brightness(0)` → `brightness(1)` with a reveal transition.

---

## 🗃️ Key Implementation Files

Just **one self-contained HTML file** with:
- Embedded CSS (Pokémon-themed, retro pixel aesthetic)
- Embedded JS (game logic, API calls, state machine)
- No frameworks, no build step — just open in browser

---

Want me to go ahead and build it now based on this plan?
