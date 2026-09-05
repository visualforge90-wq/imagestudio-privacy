# Image Studio — Store listing & Data Safety draft

Copy/adapt these into the Play Console. Nothing here is final marketing copy — tune it.

## Title (≤30 chars)
Image Studio — Photo Editor

## Short description (≤80 chars)
Private photo editor: filters, AI cutout, collage & layouts. Works offline.

## Full description (≤4000 chars)
Image Studio is a fast, private photo editor and collage maker that does its work right on
your device — your photos never leave your phone.

EDIT
• Brightness, contrast, saturation, exposure, temperature, highlights & shadows
• 9 filters, each with an adjustable strength
• One-tap Auto-enhance
• Crop & straighten with ready-made sizes for Instagram, Stories, YouTube & more
• Blur, sharpen and vignette
• Remove background with on-device AI, then drop in any color behind your subject

COMPOSE
• Collage layouts for 2–9 photos — grids, columns, feature layouts, and shaped tiles
  (diamonds, triangles, circles, pinwheel)
• Side-by-side comparisons
• Freeform canvas: stack photos, text, emoji stickers, shapes and freehand drawing;
  move, scale, rotate, opacity, undo/redo
• Backgrounds: color, blur, mosaic, gradient, pattern or your own photo

SHARE
• Export as JPG, PNG or WEBP at exact social sizes
• Save projects and reopen them anytime

Light / Dark / System themes. No account required.

Image Studio is free with ads. Unlock 4K export any time by watching a short ad.

## Category
Photography

## Content rating
Everyone (no user-generated content sharing, no objectionable material).

## Privacy policy URL
Host PRIVACY.md publicly (GitHub Pages, your site, etc.) and paste the URL here. [required]

---

# Data Safety form — draft answers

Answer these in Play Console → App content → Data safety. Based on the current build
(offline editing; ads + IAP in the published build).

**Does your app collect or share any of the required user data types?**
- If you ship **with ads (AdMob)**: **Yes** — the Google Mobile Ads SDK may collect:
  - *Device or other IDs* (advertising ID) — collected, shared, for Advertising/Marketing
    & Analytics. Not user-provided; can't be "deleted" but user can reset the ad ID.
  - *App activity / diagnostics* — as used by the ads SDK.
  These are handled by Google's SDK, not your servers. Reference Google's guidance for the
  exact AdMob Data-Safety declarations.
- Your **own** app collects/【shares】 **nothing** — no accounts, no analytics, no servers.

**Photos**: selected images are processed **on-device only** and are **not** collected or
transmitted by the app. (Do NOT declare photo "collection" — collection means sent off
device, which you don't do.)

**Is data encrypted in transit?** Yes (any ads/Play traffic uses HTTPS).

**Can users request data deletion?** No account/data on servers; users delete projects
in-app and everything is removed on uninstall.

**On-device AI note**: background removal uses ML Kit Subject Segmentation; the model is
downloaded via Google Play Services and images are processed locally (not uploaded).

> ⚠️ Data Safety answers are your legal declaration. The above reflects this build; if you
> add analytics/crash reporting/etc. later, update it. When in doubt, follow Google's
> AdMob + ML Kit Data-Safety references.

---

# Content rating (IARC) questionnaire — draft answers

Play Console → App content → Content ratings. Category: **Utility / Productivity /
Communication** (a photo editor is a tool, not a game). Typical outcome: **Everyone / PEGI 3**.

- Violence: **None**
- Sexuality / nudity: **None**
- Profanity / crude humor: **None**
- Controlled substances (drugs/alcohol/tobacco): **None**
- Gambling (real or simulated): **None**
- Fear / horror: **None**
- User-generated content shared with others in-app: **No** (there is no social feed, chat,
  or in-app sharing between users; exporting saves to the user's own device/gallery)
- Users can interact / exchange content or personal info: **No**
- Shares user location: **No**
- Digital purchases: **No** (no in-app purchases)
- Contains ads: **Yes** (banner + interstitial + rewarded, via AdMob)
- Miscellaneous / mature themes: **None**

Notes:
- Answer **"Yes" to ads** and **"No" to digital purchases** (the app has no IAP);
  mis-declaring these is a policy violation.
- Because there's no user-to-user content or communication, the rating stays at the lowest tier.
- If you later add any sharing/community feature, redo this questionnaire.

---

# Store graphics (in /store-assets, as SVG — export to PNG/JPG before uploading)

| Asset | Size | File | Play requirement |
|-------|------|------|------------------|
| App icon | 512×512 PNG | store-icon.svg | required |
| Feature graphic | 1024×500 PNG/JPG | feature-graphic.svg | required |
| Phone screenshots | 1080×1920 (or device res), 2–8 | screenshot-frame.svg (template) | min 2 |

Export SVG → PNG with Android Studio (Vector Asset), Inkscape (`inkscape -w 512 -h 512
store-icon.svg -o store-icon.png`), or any browser/online converter. Screenshots must be
real captures of the app — use screenshot-frame.svg only as a marketing wrapper if you want
captioned frames.
