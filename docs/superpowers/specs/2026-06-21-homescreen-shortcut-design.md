# Homescreen Shortcut Design

## Problem

Employees need a way to launch the Photoline Payslip web app (`index.html`, deployed via GitHub Pages) from an icon on their phone's home screen, on both iOS and Android. Tapping the icon must open the app in the phone's default browser (Safari / Chrome) — not in a fullscreen/standalone "installed app" view.

Note: neither iOS nor Android allows a website to add itself to the home screen programmatically. The employee must perform the "Add to Home Screen" gesture manually (Safari Share menu on iOS, Chrome menu on Android). This design only ensures that gesture produces a correctly named, correctly iconed, browser-opening shortcut. Instructions for employees on how to perform the gesture are out of scope — communicated separately by the requester outside the app.

## Approach

Add standard web metadata (icons + manifest) to the existing single-file `index.html`. No new app, no backend, no build step — consistent with the project's existing architecture (see `CLAUDE.md`).

Key constraint: `manifest.json` must set `"display": "browser"` and `index.html` must NOT include the `apple-mobile-web-app-capable` meta tag. Either of these would cause the OS to launch the shortcut in a standalone/fullscreen webview instead of the default browser, which is explicitly not what's wanted here.

## Assets

Source logo provided by requester (Photoline Payslip mascot logo, 1024×1024 PNG), saved at `icons/source-logo.png` for future regeneration.

Generated icon files:
- `icons/apple-touch-icon.png` — 180×180, for iOS home screen
- `icons/icon-192.png` — 192×192, for Android / manifest
- `icons/icon-512.png` — 512×512, for Android / manifest
- `icons/favicon.png` — 32×32, browser tab icon

## manifest.json

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

## index.html `<head>` additions

```html
<link rel="manifest" href="manifest.json">
<link rel="icon" href="icons/favicon.png">
<link rel="apple-touch-icon" href="icons/apple-touch-icon.png">
<meta name="apple-mobile-web-app-title" content="PL Payslip">
```

Deliberately omitted: `apple-mobile-web-app-capable` (would force iOS standalone mode).

## Out of scope

- In-app banner or instructions guiding employees through "Add to Home Screen" — requester will communicate this separately.
- Service worker / offline support / true PWA installability — not needed since the goal is browser-opening behavior, not an installed app experience.

## Cleanup

- Remove `icons/preview-small.png` (temporary verification file created during this session, not needed in the repo).
- Keep `icons/source-logo.png` as the regeneration source for future icon changes.

## Testing

Automated testing isn't meaningful for this feature (it's OS-level home-screen behavior). Verification plan:
1. Confirm all icon files render correctly and contain the expected logo content.
2. Confirm `index.html` and `manifest.json` are valid (no syntax errors) and reference correct relative paths post-deploy (GitHub Pages serves from repo root).
3. Recommend the requester manually test "Add to Home Screen" on an actual iOS and Android device after deploy, to confirm the shortcut opens in the default browser rather than standalone mode.
