# FAULTLINE — syntax diagnostic console

A single self-contained HTML file. Open it in any browser, drag a file onto it, and it tells you whether the file parses — and if not, exactly where it breaks.

Nothing is uploaded anywhere. All parsing happens locally, in your browser.

## How to use it

1. Open `faultline.html` in a browser (double-click it, no server needed).
2. Drag a `.js`, `.mjs`, `.cjs`, `.json`, `.html`, `.htm`, or `.css` file onto the drop zone — or click it to browse.
3. Read the console output below it.

A clean file prints:

```
✓ NO SYNTAX FAULTS DETECTED — clean parse across 214 lines
```

A broken one prints the line, column, error message, and a snippet with a `^` caret pointing at the exact spot.

## What it actually checks, by file type

**JS / JSON — real parsing.** JavaScript is parsed with Acorn (a real JS parser, loaded from a CDN), trying ES module syntax first and falling back to script syntax. JSON is parsed with the browser's native `JSON.parse`. Both give you a genuine syntax check — if it says clean, the file is syntactically valid.

**HTML / CSS — structural checks, not full validation.** This is the important caveat: browsers are built to be forgiving with HTML and CSS, so there's no equivalent of "throw an error" to hook into. Instead, FAULTLINE does its own structural check:

- **HTML**: tag matching — every opening tag has a matching close, closing tags match the right open tag, void elements (`<br>`, `<img>`, `<input>`, etc.) aren't required to close, and it flags a `<div>` opened but never closed, or a stray `</div>` with nothing to match.
- **CSS**: brace, parenthesis, quote, and comment balance — catches an unclosed `{`, a stray `}`, an unterminated string, or a `/*` that never got its `*/`.

What this **doesn't** catch, on purpose, because it's a different and much bigger problem than "does this parse":

- Invalid HTML nesting rules (a `<div>` inside a `<p>`) or bad attribute values
- Invalid CSS property names or values (`color: bananas;` parses fine — CSS just silently ignores rules it doesn't understand)
- Any JS/JSON *logic* bugs — wrong variable name, bad condition, off-by-one, calling a method that doesn't exist. Only things that stop the file from parsing at all get caught.

## Where it fits in a debugging workflow

Think of it as a first-pass triage step, not a replacement for actually running the code. It answers "why won't this even load" fast, before you're squinting at a blank page or a broken console. It doesn't answer "why is this behaving wrong" — that still needs real execution and testing.

## Tech notes

- Single HTML file, no build step, no server.
- JS parsing depends on Acorn loaded from cdnjs at runtime — needs an internet connection the first time (browser will cache it after).
- JSON, HTML, and CSS checking work fully offline — no external dependency.
- Everything is read via the browser's File API — files never leave your machine.
