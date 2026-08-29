# Arrangements Log — promotional site

The public marketing site for **Arrangements Log**, the private child-contact
record for iPhone and iPad.

App Store: <https://apps.apple.com/gb/app/arrangements-log/id6794839974>

---

## This is NOT the legal/support site

There are two sites, deliberately kept apart:

| Site | Repo | Serves |
|---|---|---|
| **Marketing** (this one) | `arrangements-log-site` | The promotional home page |
| **Legal & support** | `sorr535771/arrangements-log` (the `website/` folder on the SSD) | `privacy.html` and `support.html` |

**Do not move, copy or re-host the privacy policy or the support page here.**

Two reasons:

1. The shipped 1.0 binary has `https://sorr535771.github.io/arrangements-log/privacy.html`
   and `.../support.html` compiled in (`Legal.privacyPolicyURL` / `supportURL`),
   and App Store Connect has the same two URLs on file. They returned 404 once
   already, on 19 Aug 2026, after a repo transfer. They must keep serving.
2. `Tools/check.sh privacy` in the app repo gates the policy wording against
   `website/privacy.html` specifically. A second copy of the policy would drift
   from the one shown inside the app, and the gate would not catch it.

This site therefore **links out** to both pages at their canonical URLs. If you
ever change those URLs, update:

- the two footer links and the two in-body links in `index.html`
- `Legal.privacyPolicyURL` and `Legal.supportURL` in the app (a binary change —
  ship it with a version, do not do it mid-review)
- the Privacy Policy URL and Support URL fields in App Store Connect

## Files

```
index.html    the whole site — one page
style.css     all styles; palette taken from the app icon
assets/       icon at 3 sizes + web-sized screenshots
```

Assets were generated from the canonical screenshot set on the SSD
(`Screenshots 2026-08-16 listing/`, the same set uploaded to the App Store) with:

```bash
sips --resampleWidth 660 "<src>.png" --out assets/<name>.png    # iPhone shots
sips --resampleWidth 1000 "<src>.png" --out assets/ipad-home.png
```

Re-run those if the app's design changes, so the site never shows a screen the
shipped app doesn't have.

## Deliberate choices

- **No third-party requests at all.** System font stack, no Google Fonts, no CDN,
  no analytics. The page claims the app shares nothing with anyone; loading a
  webfont would hand every visitor's IP to Google while making that claim.
- **No Apple "Download on the App Store" badge.** The buttons are plain text
  links. Apple's badge artwork has its own usage rules — if you want it, take the
  official SVG from Apple's Marketing Resources and drop it inside the existing
  `<a class="btn">` elements.
- **Every claim is checked against the app.** Feature copy comes from
  `AppStore/LISTING.md` and the app README; the two quoted Analysis findings are
  examples of wording the app actually generates.
- **Light and dark**, and responsive down to 375px.

## Local preview

```bash
python3 -m http.server 8765
```

Then open <http://127.0.0.1:8765>.

## Before going live: set the domain

The site is built for a custom domain that has not been chosen yet, so the
canonical and Open Graph URLs carry a deliberate placeholder,
`REPLACE-ME.example`. It is meant to be impossible to miss. Check it is gone
before you publish:

```bash
grep -rn "REPLACE-ME.example" . --exclude-dir=.git
```

To set the real domain, in this folder:

```bash
DOMAIN=example.co.uk; sed -i '' "s/REPLACE-ME\.example/$DOMAIN/g" index.html && echo "$DOMAIN" > CNAME
```

Then point the domain's DNS at GitHub Pages and enable the custom domain in the
repository's Pages settings.

## Publishing

GitHub Pages, from the `main` branch, root folder. Push and Pages rebuilds.

## The order that matters when the domain goes live

The privacy and support links on this page still point at
`sorr535771.github.io/arrangements-log/...`, and they must keep doing so until
the new pages are actually serving. Those two URLs are compiled into the shipped
1.0 binary and held by App Store Connect, so the sequence is:

1. Put the site on the domain, with the legal and support pages reachable there.
2. Confirm both new URLs serve 200.
3. Update the four links in this page's footer and body.
4. Update the Privacy Policy URL and Support URL in App Store Connect.
5. Update `Legal.privacyPolicyURL` / `Legal.supportURL` in the app — a binary
   change, so it ships with a version, not on its own.
6. Only then let the old `github.io` pages go.

Doing 6 before 2 is what caused the 404 on 19 Aug 2026.
