---
name: ka-design
description: Grill and define the visual design system an app will be built on (design direction, color palette, typography, type scale, spacing, radius, elevation, motion, iconography, logo style) and write it to docs/DESIGN.md plus design tokens. Run once per project before the first feature, or when the visual identity changes. Part of Kansoo.
model: opus
disable-model-invocation: true
---

# ka-design

Every screen will be built from this document. Decisions made here are expensive to change later, so make them deliberately and write them down precisely enough that no one has to guess a value while building UI.

## Mode

Propose a mode with a one-sentence reason and let the user confirm:

- **think:** the user chooses the direction and every major decision; you propose options and critique.
- **ship:** you decide everything and present a summary for approval.

## 1. Context

Read `CLAUDE.md`, any PRD/spec and existing UI code or styles. Then ask only what is still unknown, 1–3 questions at a time:

- Product, audience (age, context of use), platform (web, iOS, Android, desktop)
- 3–5 words the product should feel like, and 2–3 words it must never feel like
- Competitors and how this product should stand apart from them
- References the user likes (apps, sites, Dribbble/Behance links) and what exactly they like in each
- Constraints: existing brand assets, UI library (Tailwind, shadcn, native), dark mode required, accessibility level (default WCAG 2.2 AA)

## 2. Directions

Propose **3 clearly different directions**. For each:

- Name and design movement (e.g. minimal Swiss, soft neumorphism, glassmorphism, neo-brutalism, editorial, dark neon, playful 3D)
- Mood in one sentence and why it fits the audience
- Palette preview: primary, accent, background, text (hex)
- Font pairing: heading + body
- Shape language: radius, borders, shadows
- Main risk (e.g. glassmorphism hurts contrast and performance; neon tires the eye in long sessions)

Ask the user to pick one or mix specific parts. In think mode, ask them to justify the pick in one sentence before you continue, and challenge it if it conflicts with the audience or constraints.

## 3. Grill the system

Go through each area, proposing concrete values and asking only about real choices:

- **Color:** brand primary and secondary, accent, neutral scale (50–950), semantic colors (success, warning, error, info), surface and background layers, borders, focus ring; light and dark mode values.
- **Typography:** font families with a fallback stack, weights used, type scale (display → caption) with size, line height and letter spacing per step, rules for numbers (tabular figures).
- **Spacing:** base unit and scale (e.g. 4px grid), layout grid, max content width, breakpoints.
- **Shape:** radius scale (none/sm/md/lg/full) and which components use which, border widths.
- **Elevation:** shadow scale or alternative depth system (borders, blur, tone shifts) matching the direction.
- **Motion:** durations, easing curves, what animates and what never does, reduced-motion behaviour.
- **Iconography:** library or style (outline/filled, stroke width, corner style), sizes.
- **Imagery & illustration:** style, if any.
- **Logo direction:** type (wordmark, lettermark, symbol + wordmark), style, colors, what it must avoid. Describe the direction only; do not imitate existing brands.
- **Component principles:** buttons (variants, sizes, states), inputs, cards, empty and loading states, density.
- **Voice in UI:** tone of microcopy in one or two sentences.

Use only fonts with a license that allows commercial use (e.g. Google Fonts, or ones the user owns) and say which license applies.

## 4. Verify

- Compute contrast ratios for every text/background pair and interactive state in both modes. Fix anything below WCAG 2.2 AA (4.5:1 body text, 3:1 large text and UI elements) before writing.
- Check the palette works for common color-vision deficiencies: never rely on color alone for state.
- Check every value in the doc comes from a defined scale. No one-off values.

## 5. Output

1. `docs/DESIGN.md`:
   ```markdown
   # Design System

   ## Direction — movement, mood, why

   ## Principles — 3–5 rules for design decisions

   ## Color — tokens table, light + dark, contrast notes

   ## Typography — families, license, scale table

   ## Spacing & layout

   ## Shape

   ## Elevation

   ## Motion

   ## Iconography & imagery

   ## Logo direction

   ## Components

   ## Voice

   ## Do / Don't — concrete examples
   ```
2. Tokens in the format the project uses: CSS variables, Tailwind theme config, or `design-tokens.json`. Ask if unclear.
3. Add `docs/DESIGN.md` and the tokens file to the docs map in `CLAUDE.md` with: "Read before any UI work. Use only these tokens."

Show the summary (direction, palette, fonts, radius, key decisions) and wait for approval before writing files.

## Metrics

`{"step":"design","date":"<ISO date>","mode":"think|ship","directions_shown":3,"revisions":<rounds>}`
