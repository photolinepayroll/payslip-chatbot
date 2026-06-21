# In-App Browser Gate Design

## Problem

Employees mostly receive the payslip app link inside chat apps (Messenger, Facebook, Viber). Tapping the link there opens it inside that app's embedded in-app browser ("webview"), not the phone's real Safari/Chrome. In-app browsers do not expose an "Add to Home Screen" feature at all, so the homescreen-shortcut work from `docs/superpowers/specs/2026-06-21-homescreen-shortcut-design.md` is invisible to anyone arriving this way — even though the manifest/icons themselves are correctly deployed and verified working.

This was discovered by debugging a real report ("wala akong nakitang Add to Home Screen") — confirmed root cause: the employee opened the link from a Messenger chat, not from Chrome/Safari directly.

## Approach

Add a one-time "gate" screen to `index.html` that appears only when the page detects it's running inside a known in-app browser, and only if the employee hasn't already dismissed it once on that device/browser. The gate explains the situation and offers two paths: try to open in the real browser, or continue using the chatbot right where they are (the chatbot itself works fine inside in-app browsers — only the install/shortcut capability is missing).

This is purely additive to the existing single-file `index.html` — no build step, no backend changes, consistent with the project's architecture.

## Detection

Check `navigator.userAgent` for substrings indicating a known in-app browser (case-insensitive):
- `FBAN`, `FBAV`, `FB_IAB` — Facebook and Messenger (both use Facebook's embedded webview)
- `Instagram` — Instagram
- `Viber` — Viber
- `Line/` — Line

If none match, the employee is in a real browser — skip the gate entirely, no visible change from today's behavior.

## Persistence

Important constraint: because `manifest.json` deliberately sets `"display": "browser"` (so the homescreen shortcut opens in the normal browser instead of fullscreen/standalone — see `docs/superpowers/specs/2026-06-21-homescreen-shortcut-design.md`), the page can never reliably detect "this was actually added to the home screen." That signal is exactly what `display: "browser"` disables. There is no way around this without giving up the "opens in browser, not fullscreen" requirement, which is the whole point of the feature.

Given that, the gate uses an honest approximation: once the employee clicks *either* button on the gate ("Buksan sa Browser" or "Magpatuloy dito"), set `localStorage.setItem('plPayslipGateSeen', '1')`. On every future visit, if that key is present, skip the gate unconditionally — regardless of whether they actually completed Add to Home Screen. This trades perfect accuracy for not nagging the same person repeatedly, which is what was asked for.

## Markup and behavior

Add a new `<div id="iabGate">` as the first child of `<body>`, before the existing `<div class="container">`. It is hidden by default (`display:none` in CSS) so a real-browser visit never sees it render even briefly.

Gate content (Filipino, matching the existing app's tone and `lang="fil"`):
- Photoline Payslip logo (reuse existing icon asset)
- Heading: "Buksan muna sa iyong browser"
- Body text: "Para magamit ang 'Add to Home Screen', buksan muna ito sa Chrome o Safari — hindi sa loob ng Messenger/Facebook/Viber."
- Button 1, "Buksan sa Browser": on Android (`navigator.userAgent` contains `Android`), attempt `window.location.href = 'intent://' + location.host + location.pathname + '#Intent;scheme=https;package=com.android.chrome;end'` to try to break out into Chrome directly. Below the button, always show fallback instruction text (visible whether or not the redirect attempt fires, since it's not guaranteed to work, and iOS has no equivalent trick): "Kung hindi gumana: i-tap ang ⋯ o share icon sa itaas ng screen, piliin ang 'Open in Browser'."
- Button 2, "Magpatuloy dito": sets the `plPayslipGateSeen` localStorage flag and hides `#iabGate`, revealing the existing `.container` underneath (no page reload).

A single inline `<script>` block, placed immediately after `.container`'s closing tag (synchronous, runs before paint), does:
1. If `localStorage.getItem('plPayslipGateSeen')` is set → do nothing (gate stays hidden, container shows as today).
2. Else if user agent matches none of the in-app browser signatures → do nothing (same as above).
3. Else → set `.container`'s `style.display = 'none'` and `#iabGate`'s `style.display = 'flex'`.

Button 1's click handler also calls the same "dismiss" logic as Button 2 (set the flag, hide the gate, show the container) after attempting the redirect — so if the Android redirect fails silently, the employee still lands on a working chatbot instead of a dead end.

## Out of scope

- Detecting other in-app browsers beyond the four listed (Twitter/X, Telegram, TikTok, etc.) — can be added later by extending the substring list if needed.
- Any server-side analytics on how often the gate is shown or which button is clicked.
- An equivalent "break out" trick for iOS — none exists reliably; instructions-only is the fallback there.

## Testing

Not unit-testable in the traditional sense (it's UA-sniffing + DOM visibility). Verification plan:
1. Confirm the gate is hidden by default and the existing chatbot still loads normally when no in-app browser UA and no localStorage flag are present (regression check — most employees still use real browsers directly).
2. Manually override `navigator.userAgent` in browser devtools (or use a real Messenger/Facebook in-app browser) to confirm the gate appears with correct copy and both buttons work.
3. Confirm clicking either button hides the gate, shows the chatbot, and that reloading the page afterward does not show the gate again (localStorage persisted).
4. Manually test the Android Chrome redirect attempt on a real Android device inside Messenger's in-app browser, since `intent://` behavior can vary by Android/Messenger version and isn't fully testable any other way.
