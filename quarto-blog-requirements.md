# Requirements: ML/AI Visualization Blog (Quarto + GitHub Pages)

## 1. Objective

Build a working prototype of a static technical blog focused on machine learning
and data visualization. Content is authored in Quarto, rendered to static HTML,
and deployed to GitHub Pages via GitHub Actions.

The prototype is successful when I can add a new post by creating one `.qmd`
file, running `git push`, and seeing it live — with no manual build step and no
manual image export.

## 2. Assumptions — confirm before building

Flag these back to me if any are wrong; do not silently guess.

- Language for code chunks: **Python** (matplotlib, plotly, numpy, pandas)
- Diagrams: an existing library of draw.io diagrams, to be stored as
  `.drawio.svg` files committed to the repo
- Hosting: GitHub Pages on the default `github.io` domain (no custom domain yet)
- Repo layout: blog lives at the repo root, deployed from the `gh-pages` branch
- Python environment managed with `uv` and a `pyproject.toml`; fall back to
  `venv` + `requirements.txt` if that adds friction

## 3. Stack

| Concern | Choice |
|---|---|
| Authoring / rendering | Quarto (blog project type) |
| Code execution | Python via Jupyter kernel |
| Static plots | matplotlib |
| Interactive plots | plotly |
| Diagrams | draw.io, saved as editable `.drawio.svg` |
| Hosting | GitHub Pages |
| CI | GitHub Actions |

## 4. Repository structure

Produce something close to this. Deviate if Quarto conventions demand it, but
explain why.

```
.
├── _quarto.yml
├── index.qmd                  # post listing / home
├── about.qmd
├── pyproject.toml
├── .gitignore
├── styles.css                 # custom overrides
├── posts/
│   ├── _metadata.yml          # shared post defaults
│   ├── welcome/
│   │   └── index.qmd
│   ├── attention-visualized/
│   │   ├── index.qmd
│   │   └── diagrams/
│   │       └── attention.drawio.svg
│   └── calibration-curves/
│       └── index.qmd
├── diagrams/                  # diagrams shared across posts
└── .github/
    └── workflows/
        └── publish.yml
```

## 5. Functional requirements

### 5.1 Site

- Blog listing on the home page, reverse-chronological, with categories and a
  search box
- Working RSS feed
- Light/dark theme toggle
- Per-post table of contents for long posts
- Code copy button and syntax highlighting
- LaTeX math rendering (KaTeX or MathJax), inline and display

### 5.2 Diagrams — this is the part I care most about

- Diagrams are stored as `.drawio.svg`: a single file that is both a
  browser-renderable SVG and an editable draw.io document. **No separate source
  file, no export step, no build-time conversion.**
- Per-figure display sizing. Source canvas dimensions are not standardized and
  must not need to be. Demonstrate all of these in the sample posts:
  - default column width
  - explicit width, e.g. `{width=450}`
  - full page width via `.column-page`
  - edge-to-edge via `.column-screen-inset` for one very wide diagram
- Click-to-zoom lightbox on figures, so wide diagrams remain readable
- A global CSS rule capping diagram height (`max-height` with `width: auto`) so
  tall portrait diagrams don't take over the viewport
- **Dark mode must not break diagrams.** draw.io exports transparent
  backgrounds with dark strokes, which disappear on a dark theme. Solve this and
  document the approach in the README. Acceptable solutions: force a light
  background behind figure SVGs via CSS, or use a diagram-specific CSS filter.
  Do not solve it by disabling the dark theme.
- Cross-referenceable figures (`{#fig-...}` + `@fig-...`) with captions

### 5.3 Build and deploy

- GitHub Action triggering on push to `main` that installs Quarto and the Python
  environment, renders the site, and publishes to GitHub Pages
- Use Quarto's `freeze: auto` so posts with executed code are not re-run on CI
  unless their source changed. Commit the `_freeze/` directory.
- Build must fail loudly on render errors — no publishing a half-broken site
- Local development: `quarto preview` works with live reload

## 6. Sample content

Three short sample posts, written only to exercise the machinery. Placeholder
prose is fine — do not write real technical content, I'll do that.

1. **welcome** — plain Markdown, no code, no diagrams. Baseline.
2. **attention-visualized** — one wide `.drawio.svg` architecture diagram in
   `.column-page`, one narrow diagram inline, a display-math block, and a
   cross-reference to each figure.
3. **calibration-curves** — a matplotlib figure and a plotly figure generated
   from executed Python chunks, with code folding enabled (`code-fold: true`).

If I have not supplied real `.drawio.svg` files, generate simple valid
placeholders — including the embedded draw.io XML, so I can verify they open in
the editor.

## 7. Documentation

A `README.md` covering:

- One-time setup on a new machine
- How to create a new post
- How to add or edit a diagram, including the VS Code Draw.io Integration
  extension workflow for editing `.drawio.svg` in place
- The sizing options from §5.2, with copy-pasteable snippets
- How the dark-mode diagram fix works and what to avoid when authoring diagrams
- How to convert an existing library of `.drawio` files to `.drawio.svg`

## 8. Acceptance criteria

Verify each of these and report the result. Don't claim done without checking.

- [ ] `quarto render` completes with zero errors and zero warnings
- [ ] `quarto preview` serves the site with live reload
- [ ] All three sample posts appear on the listing page in the right order
- [ ] Every diagram is visible and legible in **both** light and dark themes
- [ ] The wide diagram genuinely breaks out of the text column
- [ ] Lightbox zoom works on at least one figure
- [ ] A `.drawio.svg` file opens correctly in the draw.io editor after rendering
- [ ] Figure cross-references resolve to numbers, not literal `@fig-` text
- [ ] Math renders in both inline and display form
- [ ] The plotly chart is interactive in the built HTML, not a static image
- [ ] RSS feed is generated and well-formed
- [ ] The GitHub Action file is syntactically valid (`actionlint` or equivalent)

## 9. Out of scope

Custom domain, comments, analytics, newsletter/email capture, multi-author
support, real post content, migrating my existing diagrams.

## 10. Working style

- Ask before making structural choices not covered here; don't invent scope
- Prefer Quarto's built-in features over custom CSS/JS where both would work
- Keep `styles.css` small and commented — every rule should have a reason
- Show me the diff before large refactors
