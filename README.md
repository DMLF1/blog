# ML/Viz Notes

A Quarto blog prototype: Python code chunks (matplotlib + Plotly), diagrams
as editable `.drawio.svg` files, deployed to GitHub Pages via GitHub
Actions on every push to `main`.

## One-time setup on a new machine

1. Install [Quarto](https://quarto.org/docs/get-started/) and
   [uv](https://docs.astral.sh/uv/getting-started/installation/).
2. From the repo root:
   ```sh
   uv sync
   ```
   This creates `.venv/` with everything the posts' code chunks need
   (matplotlib, plotly, pandas, numpy, jupyter, ipykernel). Quarto's Jupyter
   engine auto-detects a `.venv`/`venv` directory next to `_quarto.yml` and
   uses it directly — no manual kernel registration needed, and the exact
   same mechanism is what CI uses (see `.github/workflows/publish.yml`).
3. **Activate the venv before running Quarto** (`source .venv/bin/activate`).
   Quarto's auto-detection finds the interpreter either way, but the
   Jupyter kernel it launches needs an activated environment to resolve its
   own dependencies (e.g. `pyyaml`) correctly on some setups.
4. Render once to confirm everything works:
   ```sh
   quarto render
   ```

## Adding a new post

1. Create a new folder under `posts/`, e.g. `posts/my-new-post/`.
2. Add an `index.qmd` with front matter (`title`, `date`, `categories`).
   Defaults shared by every post (table of contents, code folding, etc.)
   live in `posts/_metadata.yml` — only override what's different.
3. `git push`. The GitHub Action renders the site and publishes it to the
   `gh-pages` branch; nothing needs to be built locally first.

## Adding or editing a diagram

Diagrams are `.drawio.svg` files: a single file that is simultaneously a
normal browser-renderable SVG *and* an editable draw.io document (draw.io
embeds its own XML in the SVG's `content` attribute when you export with
"include a copy of my diagram" checked — nothing else reads or writes that
attribute, so the file never goes stale relative to a separate source file).

**Recommended: VS Code + the Draw.io Integration extension**
([`hediet.vscode-drawio`](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)).
With it installed, opening a `.drawio.svg` file in VS Code opens the draw.io
editor in place — edit visually, save, and the SVG on disk updates with
both the new rendering and the new embedded diagram source. Put new
diagrams next to the post that uses them (e.g.
`posts/my-post/diagrams/thing.drawio.svg`); put anything reused across
multiple posts in the top-level `diagrams/` directory instead.

**Without the extension**: use the draw.io desktop app or
[app.diagrams.net](https://app.diagrams.net), then
*File → Export as → SVG…* and check **"Include a copy of my diagram"**
before saving. Re-opening that same file (in the desktop app, the web app,
or the VS Code extension) loads the embedded diagram for editing, exactly
as if it were a native `.drawio` file.

### Sizing a diagram in a post

The source canvas size doesn't matter — pick a display size independently
with any of these:

```md
<!-- default column width -->
![Caption](diagrams/thing.drawio.svg){#fig-thing}

<!-- explicit width -->
![Caption](diagrams/thing.drawio.svg){#fig-thing width=450}

<!-- full page width -->
::: {.column-page}
![Caption](diagrams/thing.drawio.svg){#fig-thing}
:::

<!-- edge-to-edge -->
::: {.column-screen-inset}
![Caption](diagrams/thing.drawio.svg){#fig-thing}
:::
```

Reference any figure elsewhere in the post with `@fig-thing` — Quarto
resolves it to a real figure number at render time.

### Dark mode and diagrams

draw.io exports a transparent background with dark strokes and text. On a
light page that's invisible on white — fine. On Quarto's dark theme, the
page background goes dark too and the diagram effectively disappears
(dark-on-dark).

The fix, in `styles.css`:

```css
body.quarto-dark img[src$=".drawio.svg"] {
  background-color: #ffffff;
  border-radius: 4px;
  padding: 0.5rem;
}
```

`body.quarto-dark` is the class Quarto's built-in light/dark toggle actually
applies to `<body>` (confirmed by inspecting rendered output — it is *not*
a `data-bs-theme` attribute, which only appears on the navbar). The rule
forces a light backing plate behind every `.drawio.svg` image specifically,
so the diagram renders exactly as authored regardless of site theme.

A CSS color filter (e.g. `filter: invert(1)`) was considered and rejected:
it "fixes" contrast but also inverts every color you deliberately chose in
the diagram (a red "error" box becomes cyan, etc.). The backing-plate
approach is content-agnostic — it doesn't touch the diagram's own colors at
all.

**What to avoid when authoring diagrams**: don't rely on pure black
(`#000000`) text or strokes with no fill behind them if you ever plan to
show the diagram *without* the plate (e.g. embedding it somewhere this CSS
rule doesn't reach) — otherwise you're back to relying on this fix. Sticking
to draw.io's default palette (as the sample diagrams do) works well against
a white plate in both themes.

There's also a global height cap, for tall/portrait diagrams:

```css
.quarto-figure img {
  max-height: 70vh;
  width: auto;
}
```

This applies to all figures, not just diagrams — a tall diagram or plot is
scaled down to fit within 70% of the viewport height, preserving aspect
ratio.

## Converting an existing `.drawio` library to `.drawio.svg`

**One-off, via the desktop app or web app**: open the `.drawio` file,
*File → Export as → SVG…*, check "Include a copy of my diagram", save next
to (or in place of) the original.

**Batch conversion, via the draw.io CLI** (the desktop app ships a
command-line mode):

```sh
# one file
drawio -x -f svg --embed-diagram -o output.drawio.svg input.drawio

# a whole directory, preserving names
for f in path/to/library/*.drawio; do
  drawio -x -f svg --embed-diagram -o "${f%.drawio}.drawio.svg" "$f"
done
```

`--embed-diagram` is the CLI equivalent of the "include a copy of my
diagram" checkbox — without it you get a plain, non-editable SVG. Verify a
converted file round-trips by reopening it in draw.io before deleting the
original `.drawio` source.

## Local development

```sh
source .venv/bin/activate
quarto preview
```

Serves the site locally with live reload — editing any `.qmd`, `.css`, or
diagram triggers an automatic rebuild and browser refresh.

## Build and deploy

`.github/workflows/publish.yml` triggers on every push to `main`:
it installs Quarto and syncs the `uv` environment, runs `quarto render`,
and — only if that succeeds — publishes the result to the `gh-pages`
branch via `quarto-actions/publish`. A render error fails the job before
the publish step runs, so a broken build never reaches the live site.

`execute: freeze: auto` (in `_quarto.yml`) means a post's code is only
re-executed when its own source changes; otherwise the cached output in
`_freeze/` is reused. **`_freeze/` is committed to the repo** — don't
`.gitignore` it — so CI doesn't need to re-run every post's code on every
push, only the ones that actually changed.

## Repository layout

```
_quarto.yml              # site config: theme, nav, execute/freeze, format options
index.qmd                # blog listing (reverse-chronological, categories, RSS)
about.qmd
pyproject.toml           # Python deps for code chunks, managed with uv
styles.css                # small, commented overrides — see "Dark mode and diagrams"
posts/
  _metadata.yml          # defaults shared by every post (toc, code-fold, ...)
  <post-slug>/
    index.qmd
    diagrams/            # diagrams used only by this post
    references.bib       # optional: bibliography for [@key] citations
diagrams/                # diagrams shared across multiple posts
.github/workflows/publish.yml
```
