# Faultline

A single-file, offline, client-side static analysis tool for JavaScript, JSON, HTML, and CSS. Drop a file (or a whole project folder) in and get exact fault locations, an interactive structure tree, one-click fixes for common mistakes, and basic bundle metrics — all parsed in your browser. Nothing is uploaded anywhere.

Open `faultline.html` in any modern browser. No build step, no install, no server.

---

## What it does

### 1. Structure / AST Visualizer
- **JavaScript** — on a clean parse, renders the real ESTree AST (via [acorn](https://github.com/acornjs/acorn)) as a collapsible tree. On a syntax error, acorn can't hand back a tree (it throws before returning one), so Faultline falls back to a token-by-token scan and shows exactly how far it got before the file broke — the last token reached is highlighted in red.
- **JSON** — parsed with a hand-rolled recursive-descent parser (not just `JSON.parse`), so every object, array, and value in the tree carries its own line/column. Duplicate keys are flagged as warnings (still valid JSON — the last one wins) rather than errors.
- **HTML / CSS** — real parsers aren't used (browsers themselves recover from almost any HTML/CSS rather than throwing), so these get a structural tree instead: tag nesting for HTML, brace nesting for CSS, built as the file is scanned and stopped at the exact point where nesting breaks.

### 2. Context-Aware Quick Fixes
A small rule engine that only offers a fix when one is safely automatable — it does **not** try to fix everything:
- JSON: trailing commas, unquoted object keys
- JS / CSS: unbalanced brackets — since the scanner tracks exactly what's still open when parsing stops, it can append precisely the right closing characters in the right order
- HTML: unclosed tags, mismatched tags, stray closing tags

Each fix shows a before/after diff and gives you **Click to Fix** (re-scans in place), **Copy Fixed Code**, and **Download Fixed File**.

### 3. Multi-File Project Mode
Switch to "Project Folder," then drag a folder in or use the folder picker. Every supported file gets scanned individually, and Faultline builds a lightweight import graph from `import`/`require` specifiers, resolving **relative** (`./`, `../`) imports against the files you actually dropped. Anything that doesn't resolve is flagged as a broken import path, and the longest resolved chain gives you a dependency-depth number.

### 4. Performance & Bundle Metrics
For whichever file (or whole project) you last scanned:
- Estimated minified size (comment/whitespace stripped) vs. original, with a rough % savings
- Declared imports, split into relative vs. package specifiers
- Unused-import warnings (an identifier imported but never referenced again in the file)
- Duplicate JSON keys

---

## Usage

**Single file:** click or drag a `.js`/`.mjs`/`.cjs`/`.json`/`.html`/`.css` file onto the dropzone. Results stream into the Console tab; once a scan finishes, the Structure, Quick Fix, and Metrics tabs light up.

**Project folder:** click "Project Folder" above the dropzone, then drag a folder in (or use the picker). The Project tab shows a per-file status list, the import graph summary, and any broken import paths. Click any file in the list to see its own structure tree.

**Clear console** wipes the log and resets all tabs.

---

## How it's built (and what's simplified)

Everything lives in one HTML file with one external dependency — `acorn`, loaded from a CDN `<script>` tag, used only for real JS parsing/tokenizing. Everything else (the JSON parser, the HTML/CSS structural checkers, the fix engine, the import-graph resolver, the metrics) is hand-written vanilla JS, on purpose: shipping a real bundler, ESLint-via-WASM, or an editor like Monaco isn't something you can drop into a single dependency-free HTML file without a build step, so instead of faking that, Faultline implements the same *outcomes* — real structure, real fixes, real cross-file checking — with lighter, transparent engines. A few things are deliberately scoped down rather than pretended to be more than they are:

- **Quick Fix** covers specific, mechanically-safe patterns (trailing commas, unquoted keys, unbalanced brackets, unclosed/mismatched tags) — not arbitrary error correction. If there's no automatic fix for a given fault, the tab says so rather than guessing.
- **Project mode is not a real bundler.** It has no module resolution algorithm, no `node_modules` handling, no `package.json` `exports` support — it only traces relative imports against the files you dropped. Bare package imports (`import x from 'react'`) are intentionally skipped, not flagged as broken.
- **Unused-import detection** is a regex-based heuristic (checks whether an imported name appears elsewhere in the file). It can miss shadowing edge cases; it's a signal, not a guarantee.
- **HTML/CSS "structure"** reflects tag/brace nesting, not a full parsing-algorithm or property/value validator — declarations inside a CSS rule are shown as raw text rather than individually parsed.
- **Minified-size estimate** is a comment/whitespace-stripping heuristic, not a real minifier — useful as a signal, not a byte-exact number.

Large files are capped when rendering the structure tree (a few thousand nodes) so a huge file can't hang the tab; you'll see a truncation notice if that limit is hit.

## Privacy

Every file you drop is read and parsed entirely in your browser via the File API. Nothing is sent to a server — the only network request the page itself makes is fetching the `acorn` library from a CDN when the page first loads.
