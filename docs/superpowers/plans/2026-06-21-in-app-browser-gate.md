# In-App Browser Gate Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Show employees a one-time "open in your real browser" screen when they reach the payslip app through a Messenger/Facebook/Viber/Instagram/Line in-app browser, since those embedded browsers have no "Add to Home Screen" feature at all.

**Architecture:** Purely additive changes to the existing single-file `index.html` — new CSS, a hidden-by-default gate `<div>`, and a small synchronous inline `<script>` that does user-agent sniffing + localStorage persistence to decide whether to show the gate or the normal chatbot. No build step, no backend changes.

**Tech Stack:** Plain HTML/CSS/JS (vanilla, matching the rest of `index.html`). No new dependencies.

---

### Task 1: Add gate CSS rules

**Files:**
- Modify: `index.html:63-64`

- [ ] **Step 1: Insert the gate CSS rules just before `</style>`**

Change:
```html
    .system-note { font-size: 11px; color: #aaa; text-align: center; padding: 6px 14px 10px; }
  </style>
```
to:
```html
    .system-note { font-size: 11px; color: #aaa; text-align: center; padding: 6px 14px 10px; }
    #iabGate { display: none; position: fixed; inset: 0; background: #fff; z-index: 9999; flex-direction: column; align-items: center; justify-content: center; padding: 32px 24px; text-align: center; }
    #iabGate img { width: 96px; height: 96px; border-radius: 20px; margin-bottom: 16px; }
    #iabGate h2 { font-size: 18px; font-weight: 700; color: #1a1a1a; margin-bottom: 10px; }
    #iabGate p { font-size: 14px; color: #555; margin-bottom: 20px; max-width: 320px; line-height: 1.5; }
    #iabGate button { width: 100%; max-width: 280px; padding: 14px; border-radius: 24px; font-size: 15px; font-weight: 600; border: none; cursor: pointer; margin-bottom: 12px; }
    #iabGate .gate-primary { background: #1877F2; color: #fff; }
    #iabGate .gate-secondary { background: #f0f2f5; color: #333; }
    #iabGate .gate-hint { font-size: 12px; color: #999; max-width: 280px; margin-top: -4px; margin-bottom: 20px; }
  </style>
```

- [ ] **Step 2: Verify the rules were inserted**

Run:
```bash
cd "d:\HTML APP Shortcuts" && grep -c "#iabGate" index.html
```
Expected output: `8` (the 8 lines above that mention `#iabGate`).

- [ ] **Step 3: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add index.html && git commit -m "Add CSS for in-app browser gate screen"
```

---

### Task 2: Add gate HTML markup

**Files:**
- Modify: `index.html:66-67`

- [ ] **Step 1: Insert the gate markup right after `<body>`, before `<div class="container">`**

Change:
```html
<body>
  <div class="container">
```
to:
```html
<body>
  <div id="iabGate">
    <img src="icons/icon-192.png" alt="Photoline Payslip">
    <h2>Buksan muna sa iyong browser</h2>
    <p>Para magamit ang "Add to Home Screen", buksan muna ito sa Chrome o Safari — hindi sa loob ng Messenger/Facebook/Viber.</p>
    <button class="gate-primary" onclick="iabGateOpenBrowser()">Buksan sa Browser</button>
    <div class="gate-hint">Kung hindi gumana: i-tap ang &#8943; o share icon sa itaas ng screen, piliin ang "Open in Browser".</div>
    <button class="gate-secondary" onclick="iabGateDismiss()">Magpatuloy dito</button>
  </div>
  <div class="container">
```

- [ ] **Step 2: Verify the markup was inserted**

Run:
```bash
cd "d:\HTML APP Shortcuts" && grep -n "iabGateOpenBrowser\|iabGateDismiss\|id=\"iabGate\"" index.html
```
Expected output: 3 matching lines — the `id="iabGate"` div opening tag, the `onclick="iabGateOpenBrowser()"` button, and the `onclick="iabGateDismiss()"` button.

- [ ] **Step 3: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add index.html && git commit -m "Add in-app browser gate markup"
```

---

### Task 3: Add detection and button-handler script

**Files:**
- Modify: `index.html` — insert right after the `.footer` div's closing tag, before the existing main `<script>` tag.

The current relevant section (after Tasks 1-2 are applied, line numbers will have shifted down, but the surrounding text below is unique and easy to locate — search for `class="footer"`):
```html
  <div class="footer">© 2026 PHOTOLINE &nbsp;|&nbsp; Developed by GJA (HR)</div>

  <script>
    const SHEET_URL = 'https://script.google.com/macros/s/AKfycbwoZifp2RQip9Fmb6hfzJc2eAnbURfKsL9xF3g4H0cqsHS4dTLCmiCyFJWaBFaQgOaI/exec';
```

- [ ] **Step 1: Insert the gate script between the footer div and the main script tag**

Change:
```html
  <div class="footer">© 2026 PHOTOLINE &nbsp;|&nbsp; Developed by GJA (HR)</div>

  <script>
    const SHEET_URL = 'https://script.google.com/macros/s/AKfycbwoZifp2RQip9Fmb6hfzJc2eAnbURfKsL9xF3g4H0cqsHS4dTLCmiCyFJWaBFaQgOaI/exec';
```
to:
```html
  <div class="footer">© 2026 PHOTOLINE &nbsp;|&nbsp; Developed by GJA (HR)</div>

  <script>
    function iabGateDismiss() {
      localStorage.setItem('plPayslipGateSeen', '1');
      document.getElementById('iabGate').style.display = 'none';
      document.querySelector('.container').style.display = '';
      document.querySelector('.footer').style.display = '';
    }
    function iabGateOpenBrowser() {
      if (/Android/i.test(navigator.userAgent)) {
        window.location.href = 'intent://' + location.host + location.pathname + '#Intent;scheme=https;package=com.android.chrome;end';
      }
      iabGateDismiss();
    }
    (function () {
      if (localStorage.getItem('plPayslipGateSeen')) return;
      var ua = navigator.userAgent.toLowerCase();
      var iabSignatures = ['fban', 'fbav', 'fb_iab', 'instagram', 'viber', 'line/'];
      var isInApp = iabSignatures.some(function (sig) { return ua.indexOf(sig) !== -1; });
      if (!isInApp) return;
      document.querySelector('.container').style.display = 'none';
      document.querySelector('.footer').style.display = 'none';
      document.getElementById('iabGate').style.display = 'flex';
    })();
  </script>

  <script>
    const SHEET_URL = 'https://script.google.com/macros/s/AKfycbwoZifp2RQip9Fmb6hfzJc2eAnbURfKsL9xF3g4H0cqsHS4dTLCmiCyFJWaBFaQgOaI/exec';
```

Note: this creates a second, separate `<script>` block dedicated to the gate logic, immediately followed by the existing main `<script>` block (unchanged). Do not merge them — keeping the gate logic in its own block keeps it isolated from the chatbot's existing code.

- [ ] **Step 2: Verify the script was inserted correctly**

Run:
```bash
cd "d:\HTML APP Shortcuts" && grep -n "function iabGateDismiss\|function iabGateOpenBrowser\|plPayslipGateSeen\|iabSignatures" index.html
```
Expected output: 6 matching lines (the two function declarations, the two `localStorage` calls referencing `plPayslipGateSeen`, and the two `iabSignatures` references — the array declaration and its use in `.some(...)`).

- [ ] **Step 3: Sanity-check the file still parses as well-formed enough HTML (no obviously broken tags)**

Run:
```bash
cd "d:\HTML APP Shortcuts" && python3 -c "
content = open('index.html', encoding='utf-8').read()
assert content.count('<script>') == content.count('</script>'), 'mismatched script tags'
assert content.count('<style>') == content.count('</style>'), 'mismatched style tags'
assert content.count('<div') >= content.count('</div>'), 'more closing divs than opening divs'
print('tag counts OK:', content.count('<script>'), 'script blocks,', content.count('<div'), 'div opens,', content.count('</div>'), 'div closes')
"
```
Expected output: a line starting with `tag counts OK:` and no assertion error. (This won't catch every possible mistake, but it catches the most common copy-paste errors like a missing closing tag.)

- [ ] **Step 4: Commit**

```bash
cd "d:\HTML APP Shortcuts" && git add index.html && git commit -m "Add in-app browser detection and gate dismiss logic"
```

---

### Task 4: Regression check — confirm normal browser behavior is unchanged

**Files:** None modified — this is a verification-only task.

- [ ] **Step 1: Confirm the gate stays hidden and the chatbot is reachable when there's no in-app browser signature and no localStorage flag**

Run:
```bash
cd "d:\HTML APP Shortcuts" && python3 -c "
content = open('index.html', encoding='utf-8').read()
assert '#iabGate { display: none' in content, 'gate is not hidden by default in CSS — this would break normal browser visits'
print('gate is hidden by default: OK')
"
```
Expected output: `gate is hidden by default: OK`

- [ ] **Step 2: Confirm the forbidden standalone-mode tag still hasn't crept in**

Run:
```bash
cd "d:\HTML APP Shortcuts" && grep -c "apple-mobile-web-app-capable" index.html
```
Expected output: `0`

---

### Task 5: Push and manually verify

- [ ] **Step 1: Push all commits to GitHub**

```bash
cd "d:\HTML APP Shortcuts" && git push
```
Expected output: shows local branch `main` updated to remote, no errors. GitHub Pages will redeploy automatically.

- [ ] **Step 2: Manually verify normal-browser behavior is unaffected**

Not automatable — open the deployed GitHub Pages URL directly in a regular Chrome or Safari (not through Messenger/Facebook/Viber). Confirm the page looks exactly as before — no gate screen, chatbot loads immediately, no errors in the browser console (open DevTools → Console to check).

- [ ] **Step 3: Manually verify the gate appears inside a real in-app browser**

Not automatable — from an actual phone, send the deployed URL to yourself in Messenger (or Facebook/Viber) and tap it so it opens in that app's in-app browser. Confirm:
- The "Buksan muna sa iyong browser" gate appears with the Photoline Payslip icon, heading, and both buttons.
- Tapping "Magpatuloy dito" hides the gate and shows the normal chatbot, still inside the in-app browser.
- Reloading/reopening the same link afterward does NOT show the gate again (localStorage remembered the dismissal).
- On an Android phone specifically, tapping "Buksan sa Browser" either successfully opens Chrome with the page loaded, or (if that fails) the chatbot still becomes usable right there since the dismiss logic always runs afterward.
