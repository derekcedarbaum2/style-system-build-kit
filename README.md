# style-system--build-kit

[![CI](https://github.com/derekcedarbaum2/brand-system-kit/actions/workflows/ci.yml/badge.svg)](https://github.com/derekcedarbaum2/brand-system-kit/actions/workflows/ci.yml)
&nbsp;![Node 18+](https://img.shields.io/badge/node-18%2B-3E5C8A)
&nbsp;![License: MIT](https://img.shields.io/badge/license-MIT-1F6E7A)

**A style system for business documents. One you can run, not just read.**

It's for the documents that corporate product, program, sales, and operations teams ship: PRDs, strategy briefs, program schedules, status reports, one-pagers, decision memos, and decks. Most house styles are a PDF nobody opens and nothing enforces, so colors drift, the wrong blue ships, and a report looks off without anyone able to say why. This kit makes the rules executable: define your style once, in one file, and the tooling keeps every document on-style or fails the build.

What's inside:

- **One source of truth.** Every color and font lives in a single `tokens.json`. The CSS and the docs are generated from it, so they can't drift apart.
- **A linter that blocks off-style output.** Banned words, gradients, forbidden colors, and one profile's name leaking into another's document fail the check; off-token hex and box-shadow warn.
- **An accessibility gate.** Every palette is checked against WCAG AA contrast. Computed, not eyeballed.
- **A render step.** It turns any document into an image so you (or an AI agent) can judge what a rule can't catch: whether it actually looks right.
- **Document templates.** Ready-to-fill HTML for the common deliverables. Copy one, point it at your style, fill it in.

Zero dependencies. Plain Node 18+. Any agent (Codex, Cursor, Claude Code) can scaffold a new style profile by following the interview in [`docs/brand-interview.md`](docs/brand-interview.md). Claude Code also gets it as a `/brand-init` skill.

It ships with two working styles, **Northwind** and **Graphite**, so the whole pipeline runs the moment you clone it. They're deliberately different (see [the one idea](#the-one-idea) below), which makes it easy to see what the kit does. Replace them with yours.

## Preview

The same kit shown in two registers. Left: Northwind (warmth: light, serif, editorial accent). Right: Graphite (restraint: near-black, no extra color, mono labels).

| Northwind (warmth) | Graphite (restraint) |
|---|---|
| ![Northwind sample](docs/sample-northwind.png) | ![Graphite sample](docs/sample-graphite.png) |

One diagram, re-skinned to each style by swapping the token profile, nothing hand-edited:

| `--palette northwind` | `--palette graphite` |
|---|---|
| ![Persona, Northwind](docs/diagram-northwind.png) | ![Persona, Graphite](docs/diagram-graphite.png) |

> This is the structure, not a finished style. There's no "right" palette here; the kit keeps whatever style you define consistent and enforced.

> **Using an AI agent to set this up?** Point it at [`AGENTS.md`](AGENTS.md): step-by-step install, verification, and scaffolding instructions written to be followed literally.

---

## The one idea

*Why the kit ships two demo styles, and why they look nothing alike.*

> **Match the visual register to your reader.**

One discipline, two registers, because two kinds of reader judge a document by opposite signals:

| Register | Reader | Reads as wrong when | Trust comes from |
|---|---|---|---|
| **Restraint** | Technical or expert reviewer (engineers, evaluators, program reviews) | It looks polished or salesy | Near-black, no extra color, mono labels. The work speaks for itself. |
| **Warmth** | Executive, stakeholder, or customer | It looks cold or hard to scan | Light background, serif headlines, one editorial accent. Clear and credible. |

Both registers come from the same discipline (argument first, evidence over decoration, hierarchy through restraint), aimed at readers who judge differently. If your work reaches both kinds of reader, you need both registers, which is why the kit is multi-profile.

---

## Architecture: generic core + profiles

*Where everything lives. Four parts: the prose framework (`system/`), the executable tooling (`tooling/`), the styles (`profiles/`), and the templates and diagrams you fill in (`templates/`, `diagrams/`).*

```
brand-system-kit/
  system/                  # the style guides (generic, prose)
    identity.md            #   philosophy + the one idea (register and reader)
    typography.md  components.md  spacing.md  motion.md
    color-system.md  tokens-screen.md  tokens-print.md  tokens-email.md
    voice-and-tone.md  anti-patterns.md  lint-rules.md  imagery.md
    data-viz.md  slide-layouts.md  accessibility.md  medium-guide.md  README.md
    print-layout.md        #   HTML to PDF mechanics: full-bleed bg, sheet pagination, no-split cards
  tooling/                 # the executable system (zero-dep Node 18+)
    build-tokens.mjs       #   tokens.json to CSS :root block + Markdown palette table
    contrast-check.mjs     #   WCAG 2.1 AA, computed from tokens.json (mode-aware)
    lint.mjs               #   gate a document: fails on banned words, gradients, forbidden colors, name leakage; warns on off-token hex
    render.mjs             #   HTML to PNG (headless Chrome) for visual review
    brand-qa.mjs           #   one command: contrast + lint + optional render
    tokenize-svg.mjs       #   hardcoded-hex SVG to var()-driven, re-skinnable per profile
  profiles/
    northwind/             # DEMO style (warmth register). Your styles become siblings here.
      tokens.json          #   the single source of truth
      brand.css  color-system.md  sample.html   # all generated/derived
    graphite/              # DEMO style (restraint register), same file set
  templates/               # ready-to-fill HTML for common deliverables
  diagrams/                # 37 token-driven SVG templates (re-skin to any profile)
  docs/brand-interview.md  # tool-agnostic interview an agent runs to scaffold a style
  skill/brand-init/        # Claude Code wrapper around docs/brand-interview.md
  examples/bad.html        # a document full of violations, to see the linter bite
```

`system/` is the prose framework (read these for the reasoning). `tooling/` enforces it. `profiles/` are the styles. `templates/` and `diagrams/` are the things you copy and fill in.

Each style is a **profile**, which is one `tokens.json`. The tooling is style-agnostic; it never hard-codes a palette.

---

## Quickstart

*See the whole pipeline work in about a minute. Clone the repo and run these four commands from its root against the bundled demo style. No install. You should get three passes and one deliberate failure (the linter catching a bad file).*

```bash
# 1. Check the demo palette is accessible (it is)
node tooling/contrast-check.mjs profiles/northwind

# 2. Generate CSS + palette table from the tokens
node tooling/build-tokens.mjs profiles/northwind/tokens.json profiles/northwind/brand.css --md profiles/northwind/color-system.md

# 3. Gate a document (contrast + lint), and render it for the eye test
node tooling/brand-qa.mjs profiles/northwind/sample.html profiles/northwind --render

# 4. See the linter bite
node tooling/lint.mjs examples/bad.html profiles/northwind   # exits 1
```

`render.mjs` needs Chrome or Chromium. Set `CHROME_PATH` if it isn't auto-found.

The same gates run as npm scripts: `npm run build` (regenerate every profile's CSS), `npm run contrast` (check every profile), `npm run qa` (gate + render the Northwind sample).

---

## Make a document

*The fastest path to a finished deliverable.* Copy a file from [`templates/`](templates/), change its one stylesheet link to your profile's `brand.css`, replace the `{{PLACEHOLDER}}` text, preview it as a PNG with `render.mjs`, and print the PDF with headless Chrome's `--print-to-pdf` (see `system/print-layout.md`). The templates cover the common business documents:

| Template | For |
|---|---|
| `prd.html` | Product requirements: problem, scope, requirements, milestones |
| `strategy-brief.html` | A position and the reasoning behind it |
| `program-schedule.html` | Phases, milestones, and a timeline |
| `status-report.html` | Period status: health, metrics, risks, next steps |
| `one-pager.html` | A single-page overview or capability statement |
| `decision-memo.html` | A recommendation and the options behind it |

Run every template through the gate before you ship it: `node tooling/brand-qa.mjs templates/prd.html profiles/<your-style> --render`.

---

## Make your own style

*Let an agent interview you, or copy the demo and edit by hand.*

- **With any agent (Codex, Cursor, Claude Code):** have it follow [`docs/brand-interview.md`](docs/brand-interview.md). It asks about your readers, picks the register, then writes a WCAG-passing `tokens.json`, generates the CSS, and renders a sample. In Claude Code, run `/brand-init`.
- **By hand:** copy `profiles/northwind/` to `profiles/<your-style>/`, edit `tokens.json`, and re-run the Quickstart commands against your new profile.

Then wire `brand-qa.mjs` into whatever produces your documents (a docs build, a deck generator, a CI step) so nothing ships ungated.

---

## Design tokens

Tokens are [DTCG format](https://www.designtokens.org/) (W3C Design Tokens, first stable spec). `tokens.json` is the only place hex lives. CSS and docs are generated from it, so they can't drift. The `lint` group carries non-visual constants (style name, forbidden hexes, accent budget, foreign-name list) that the gate reads.

## What it is, and what it isn't

It's opinionated on purpose. The default look is minimal and text-forward, which is right for documents that have to be read and trusted, and wrong for a consumer or retail brand that lives on imagery and big color. If that's you, this isn't your tool.

It also isn't a full brand identity system. There's no logo, packaging, social, or app-UI guidance. It's the style layer for business documents: tokens, document components, and the gate that keeps them consistent.

No fonts are bundled (the demos load them from Google Fonts; swap freely). No framework. It's plumbing.

## License

MIT. See [LICENSE](LICENSE).
