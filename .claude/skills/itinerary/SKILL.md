---
name: itinerary
description: Generate a Material Design 3 dark-themed HTML travel itinerary from a trip description, save it to /itineraries/, and optionally commit + publish via GitHub Pages. Use when the user wants to plan a trip, build an itinerary page, or duplicate the NYC itinerary format for a new destination.
---

# Itinerary Generator Skill

Turns a trip description into a polished, mobile-friendly HTML itinerary that matches the visual style of the NYC reference page.

## When to invoke

Trigger this skill when the user asks to:
- "Make me an itinerary for [destination]"
- "Plan a trip to [place]"
- "Create another itinerary page"
- "Use the NYC template for [new trip]"

## Inputs to gather

Before generating, confirm (use AskUserQuestion if any are missing):

1. **Destination + trip name** (e.g. "Paris weekend", "Tokyo Oct 2026")
2. **Travel dates** (arrival → departure)
3. **Hotel / lodging** (name, address, neighborhood, any prepay/hold notes)
4. **Day-by-day plan** — for each day:
   - A short theme/title (e.g. "Inbound Hub", "Midtown Matrix")
   - 3–8 timeline blocks with: time window, activity name, strategy/notes
5. **Logistical strategy note** (optional one-paragraph "why this plan works")
6. **Transit color hints** (optional — e.g. specific subway/metro line badges)

If the user says "just generate something reasonable" or gives only a destination, fill in defaults from general knowledge of the city and confirm at the end.

## Generation steps

1. **Read the template:** `.claude/skills/itinerary/template.html` — it's the canonical structure.
2. **Pick a filename:** kebab-case, dated. Examples: `paris-jul-2026.html`, `tokyo-oct-2026.html`. Save to `itineraries/<filename>`.
3. **Replace the following sections** of the template, preserving all CSS, fonts, and component classes:
   - `<title>` — set to "<Trip Name> - <User First Name>" or similar.
   - The `.md-app-bar__title` text (e.g. "NYC Master Plan" → "Paris Master Plan").
   - The primary "Hotel" card: hotel name, address, neighborhood, payment line, date chip.
   - The "Logistical Strategy Note" card body.
   - Each `.section-title` + day card: re-label the day headers and replace the `.timeline-item` blocks with the new day's plan.
4. **Keep the design tokens** (`:root` CSS variables, `.md-card`, `.timeline`, `.subway-badge`, etc.) untouched. Only the human-readable text and per-trip transit badge colors should change.
5. **Subway badges:** Reuse the `.subway-badge` span for any transit lines. Inline-style the background/text color to match the real line color when known (NYC yellow `#FCCC0A`, NYC red `#EE352E`, London tube colors, Paris metro colors, Tokyo line colors, etc.). Keep the badge inline inside the notes sentence.

## Hub page

After saving a new itinerary, update `index.html` to be a hub listing all trips. The hub should:
- Reuse the same MD3 dark styling (copy `:root`, `body`, `.md-app-bar`, `.md-container`, `.md-card` rules from `template.html`).
- Show one `.md-card` per file in `itineraries/`, each linking to its page with trip name, dates, and destination.
- Sort newest-first by trip date.

If `index.html` is still the raw NYC page (not yet a hub), convert it on the first run: move the NYC content into `itineraries/nyc-may-2026.html` (if not already there) and replace `index.html` with the hub.

## Publishing (optional — ask first)

If the user asks to publish/host/share:
1. `git add` the new file(s), commit with message like `Add <Trip Name> itinerary`.
2. Push to `main` (this repo's GitHub Pages source).
3. Give them both links:
   - Hub: `https://jiaweiscript.github.io/Itinerary-/`
   - This trip: `https://jiaweiscript.github.io/Itinerary-/itineraries/<filename>`
4. If Pages isn't yet enabled, remind them to flip it on at Settings → Pages → Source: `main` / root.

Do not commit/push without explicit user confirmation.

## Style guardrails

- Keep the timeline tone strategic and concise — each note explains the *why* (avoid crowds, beat traffic, save a transit transfer), not just the *what*.
- Don't add emojis to the page (Font Awesome icons only, matching the template's existing `<i class="fa-...">` usage).
- Don't introduce new color tokens — work within the existing `:root` variables.
- Keep everything in a single self-contained HTML file (inline `<style>`, CDN links for fonts/icons). No build step, no external CSS files.
