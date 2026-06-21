# Homescreen Shortcut Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the Photoline Payslip web app installable as a home-screen icon on iOS and Android that opens in the phone's default browser (not standalone/fullscreen mode).

**Architecture:** Static asset additions only — generate icon images from the existing source logo, add a `manifest.json` with `"display": "browser"`, and add icon/manifest `<link>`/`<meta>` tags to `index.html`. No build step, no backend changes, no JS logic changes.

**Tech Stack:** Plain HTML, Pillow (Python) for one-off image resizing, GitHub Pages for deployment.

---

### Task 1: Generate icon image files

**Files:**
- Create: `icons/apple-touch-icon.png` (180×180)
- Create: `icons/icon-192.png` (192×192)
- Create: `icons/icon-512.png` (512×512)
- Create: `icons/favicon.png` (32×32)
- Input: `icons/source-logo.png` (already in repo, 1024×1024)

- [ ] **Step 1: Generate all four sizes from the source logo**

Run:
```bash
cd "d:\HTML APP Shortcuts" && python3 -c "
from PIL import Image
src = Image.open('icons/source-logo.png').convert('RGB')
sizes = {
    'icons/apple-touch-icon.png': 180,
    'icons/icon-192.png': 192,
    'icons/icon-512.png': 512,
    'icons/favicon.png': 32,
}
for path, size in sizes.items():
    src.resize((size, size), Image.LANCZOS).save(path)
    print(path, 'written')
"
```
Expected output: four lines, one per file, each ending in `written`.

- [ ] **Step 2: Verify each file exists with the correct dimensions**

Run:
```bash
cd "d:\HTML APP Shortcuts" && python3 -c "
from PIL import Image
expected = {
    'icons/apple-touch-icon.png': 180,
    'icons/icon-192.png': 192,
    'icons/icon-512.png': 512,
    'icons/favicon.png': 32,
}
for path, size in expected.items():
    im = Image.open(path)
    assert im.size == (size, size), f'{path} is {im.size}, expected {(size,size)}'
    print(path, im.size, 'OK')
"
```
Expected output: four lines ending in `OK`. If any line raises `AssertionError`, re-run Step 1.

- [ ] **Step 3: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add icons/apple-touch-icon.png icons/icon-192.png icons/icon-512.png icons/favicon.png && git commit -m "Add generated homescreen icon assets"
```

---

### Task 2: Create manifest.json

**Files:**
- Create: `manifest.json` (repo root)

- [ ] **Step 1: Write the manifest file**

Create `d:\HTML APP Shortcuts\manifest.json` with exactly this content:

```json
{
  "name": "Photoline Payslip",
  "short_name": "PL Payslip",
  "display": "browser",
  "start_url": "./index.html",
  "icons": [
    { "src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

- [ ] **Step 2: Validate the JSON is syntactically correct**

Run:
```bash
cd "d:\HTML APP Shortcuts" && python3 -c "
import json
data = json.load(open('manifest.json'))
assert data['display'] == 'browser'
assert data['short_name'] == 'PL Payslip'
assert len(data['icons']) == 2
print('manifest.json valid:', data)
"
```
Expected output: a line starting with `manifest.json valid:` followed by the parsed dict. If it raises an error, fix `manifest.json` and re-run.

- [ ] **Step 3: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add manifest.json && git commit -m "Add web manifest for homescreen shortcut"
```

---

### Task 3: Wire up icon/manifest tags in index.html

**Files:**
- Modify: `index.html:6` (immediately after the `<title>` line)

- [ ] **Step 1: Insert the manifest and icon tags**

In `index.html`, change:

```html
  <title>Payslip Inquiry</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
```

to:

```html
  <title>Payslip Inquiry</title>
  <link rel="manifest" href="manifest.json">
  <link rel="icon" href="icons/favicon.png">
  <link rel="apple-touch-icon" href="icons/apple-touch-icon.png">
  <meta name="apple-mobile-web-app-title" content="PL Payslip">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
```

Do NOT add an `apple-mobile-web-app-capable` meta tag — it would force iOS into standalone/fullscreen mode instead of opening Safari.

- [ ] **Step 2: Verify the tags were inserted and the forbidden tag is absent**

Run:
```bash
cd "d:\HTML APP Shortcuts" && grep -n "manifest.json\|apple-touch-icon\|apple-mobile-web-app-title\|icons/favicon" index.html
```
Expected output: 4 matching lines (manifest link, favicon link, apple-touch-icon link, apple-mobile-web-app-title meta).

Run:
```bash
cd "d:\HTML APP Shortcuts" && grep -c "apple-mobile-web-app-capable" index.html
```
Expected output: `0`

- [ ] **Step 3: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add index.html && git commit -m "Add manifest and icon links to index.html head"
```

---

### Task 4: Clean up temporary files

**Files:**
- Delete: `icons/preview-small.png` (temporary verification file, not needed in the repo)
- Keep: `icons/source-logo.png` (regeneration source — do not delete)

- [ ] **Step 1: Remove the temporary preview file**

```bash
cd "d:\HTML APP Shortcuts" && rm icons/preview-small.png
```

- [ ] **Step 2: Verify it's gone and source-logo.png is still present**

Run:
```bash
cd "d:\HTML APP Shortcuts" && ls icons/
```
Expected output: `apple-touch-icon.png  favicon.png  icon-192.png  icon-512.png  source-logo.png` (no `preview-small.png`).

- [ ] **Step 3: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add -A icons/ && git commit -m "Remove temporary icon preview file"
```

---

### Task 5: Push and manually verify on real devices

- [ ] **Step 1: Push all commits to GitHub**

```bash
cd "d:\HTML APP Shortcuts" && git push
```
Expected output: shows local branch `main` updated to remote, no errors. GitHub Pages will redeploy automatically.

- [ ] **Step 2: Wait for GitHub Pages to redeploy, then manually verify on an iOS device**

Not automatable — requires a physical device. Open the deployed GitHub Pages URL in Safari on an iPhone, tap Share → Add to Home Screen, confirm the icon shows the Photoline Payslip logo and the label reads "PL Payslip", then tap the new home screen icon and confirm it opens in Safari (visible address bar / Safari UI), not in a fullscreen app-like view.

- [ ] **Step 3: Manually verify on an Android device**

Open the deployed URL in Chrome on an Android phone, open the Chrome menu (⋮) → Add to Home Screen, confirm the same icon/label, then tap the new icon and confirm it opens in Chrome (visible address bar), not in a standalone window.
