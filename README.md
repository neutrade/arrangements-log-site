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

## Domain and DNS

The site is built for **childarrangements.com** (registered 29 Aug 2026, GoDaddy).
The app keeps its own name — Arrangements Log — the domain is just the address.

`CNAME` holds the apex, so GitHub serves the site at `childarrangements.com` and
redirects `www` to it. At GoDaddy, replace the parked records with:

| Type | Name | Value |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | sorr535771.github.io |

Delete the GoDaddy parking A records first (currently 15.197.148.33 and
3.33.130.190) or they will fight the new ones. Then in the repository's
Settings → Pages, set the custom domain to `childarrangements.com` and tick
**Enforce HTTPS** once the certificate is issued (it can take up to an hour).

Check it has taken:

```bash
dig +short childarrangements.com A
curl -sI https://childarrangements.com | head -1
```

## Publishing

GitHub Pages, from the `main` branch, root folder. Push and Pages rebuilds.

## This site now hosts its own privacy and support pages

`privacy.html` and `support.html` here carry the SAME wording as
`website/privacy.html` in the other repo — copied verbatim, because that wording
is gated against `Legal.swift` by `Tools/check.sh privacy`. Only the surrounding
page furniture differs.

That means the policy now exists in three places. Until the switch-over below is
finished, **the copy in the old `website/` folder remains the canonical one** —
edit that first, then mirror the change here.

## Switch-over order, when the domain is live

The old pages at `sorr535771.github.io/arrangements-log/...` are still what the
shipped 1.0 binary and App Store Connect point at. Do not touch them until the
new ones are serving:

1. Push this repo, set the custom domain, confirm
   `https://childarrangements.com/privacy.html` and `/support.html` both return 200.
2. Update the Privacy Policy URL and Support URL in App Store Connect.
3. Update `Legal.privacyPolicyURL` / `Legal.supportURL` in the app — a binary
   change, so it ships with a version (1.0.1), never on its own.
4. Point `Tools/check.sh`'s `$site` path at this repo's `privacy.html`, so the
   parity gate follows the canonical copy.
5. Only once a build carrying the new URLs is live may the old pages go.

Doing 5 before 1 is what caused the 404 on 19 Aug 2026.
