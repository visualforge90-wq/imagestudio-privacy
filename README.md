# Image Studio

A native Android image editor & collage app built with Kotlin, Jetpack Compose and
Material 3. This repository is a **runnable full scaffold with a working core flow** —
the architecture, rendering engine, export pipeline, and the primary editing surfaces
are implemented; a few advanced features are wired but stubbed (see Roadmap).

---

## Opening the project

1. Open **Android Studio** (Koala / 2024.1+ recommended) → *Open* → select this folder.
2. Let Gradle sync. Android Studio will download the Gradle distribution declared in
   `gradle/wrapper/gradle-wrapper.properties` (Gradle 8.9) and the dependencies.
3. Run the `app` configuration on a device or emulator (min SDK 24 / Android 7.0).

> **Note on the Gradle wrapper JAR:** to keep the archive small, `gradle/wrapper/gradle-wrapper.jar`
> and the `gradlew` / `gradlew.bat` scripts are **not** bundled. Android Studio regenerates
> them automatically on first sync. If you build from the command line instead, run
> `gradle wrapper` once (using a locally installed Gradle) to generate them, then use `./gradlew`.

---

## Architecture

Clean, layered, and framework-light so it stays readable:

```
Presentation (Compose screens)
        │  observes StateFlow
   ViewModel (per feature)
        │  calls
     Domain models  ── pure, serializable state (no Android deps)
        │  rendered by
      Data layer  ── BitmapLoader · CanvasCompositor · MediaStoreExporter
```

Key ideas:

- **Non-destructive editing.** Nothing ever mutates source pixels. An edit is described
  by `EditState` (source Uri + `Transform` + `Adjustments` + `FilterPreset`). Layouts are
  described by `SideBySideConfig` / `CollageConfig`. Pixels are produced only when a preview
  or export is needed.
- **One render path = WYSIWYG.** `CanvasCompositor` turns state into a `Bitmap` for export and
  thumbnails (off the main thread, on `Dispatchers.Default`). Collage and side-by-side share the
  exact same drawing code (`LayoutRenderer`) with their interactive previews, which draw *live* on
  the Compose canvas — so dragging/pinching a cell repaints instantly at 60fps instead of
  re-compositing through the exporter, while staying pixel-faithful to the export.
- **Shared color math.** `ColorMatrixFactory` builds one 4×5 color matrix from the filter +
  adjustments; it drives both preview and export identically.
- **Manual DI.** `AppContainer` in `ImageStudioApp` provides the loader/compositor/exporter.
  Swap for Hilt/Koin as the app grows.

### Package map

| Package | Contents |
|---|---|
| `domain.model` | `EditModels`, `LayoutModels`, `Project` — serializable state + built-in collage templates |
| `data.image` | `BitmapLoader` — downsampled, EXIF-aware, LRU-cached decoding |
| `data.render` | `CanvasCompositor` — renders edit / side-by-side / collage to bitmaps |
| `data.export` | `MediaStoreExporter` — saves to `Pictures/ImageStudio` via MediaStore |
| `util` | `ColorMatrixFactory` — adjustment + filter matrices |
| `ui.theme` | Material 3 theme, dynamic color, dark/light |
| `ui.navigation` | Routes + `NavHost` |
| `ui.home` | Dashboard |
| `ui.editor` / `ui.sidebyside` / `ui.collage` | Feature screens + ViewModels |
| `ui.freeform` / `ui.projects` | Wired stubs |
| `ui.components` | Photo picker, sliders, export sheet, top bar |

---

## What works today

- **Home dashboard** with Material 3 cards + dynamic color, dark/light.
- **Photo import** via the Android Photo Picker (no storage permission required).
- **Photo editor** (`ui.editor`):
  - Adjustments: brightness, contrast, saturation, exposure, temperature, highlights, shadows (live preview).
  - 9 filter presets (B&W, Vintage, Warm, Cool, Dramatic, Fade, Bright, High-contrast, Original), each with a **strength slider**, plus **one-tap Auto-enhance**.
  - Transforms: rotate 90°, flip horizontal / vertical.
  - **Remove background** (Cutout tab): on-device AI isolates the subject; pick a backdrop color
    or keep it transparent (export PNG).
  - **Crop & straighten**: **social crop presets** (1:1, 4:5 IG post, 3:4 IG tall, 9:16 Story/Reel, 16:9, 2:3 Pinterest); drag the corners or interior of an interactive overlay (with
    rule-of-thirds grid), a straighten slider (±15°, auto-cropped to the largest inscribed
    rectangle so there are no empty corners), and quick aspect snaps (1:1, 4:5, 16:9, Free).
  - **Effects**: blur (fast separable box blur), sharpen (unsharp mask), and vignette. Effect
    strength scales with the working resolution, so the preview matches the export.
  - **Undo / redo** history and **reset**.
  - **Press-and-hold to peek the "before" image.**
- **Side-by-Side** (`ui.sidebyside`): pick two photos, vertical/horizontal split, ratio presets,
  spacing, corner radius, divider, before/after labels, and **Color / Blur / Mosaic / Gradient / Pattern / Photo backgrounds**. **Drag each side to
  reposition and pinch to zoom** its image within the frame.
- **Collage** (`ui.collage`): multiple layouts for every photo count from 2 to 9 (grids, columns,
  rows, feature layouts, and **shaped layouts** — diamonds, triangles, circles, and a real
  pinwheel via non-rectangular polygon/ellipse cells), aspect-ratio presets, spacing, corner
  radius, and **Color / Blur / Mosaic / Gradient / Pattern / Photo backgrounds** (blur is an
  Instagram-style blur of your first photo; patterns are drawn procedurally; photo uses one of your images).
  **Drag any cell to reposition and pinch to zoom** its photo, with a Reset-framing control.
- **Freeform canvas** (`ui.freeform`): multi-layer editing — add image / text / **emoji sticker** /
  shape layers and freehand drawing; move, pinch-scale and rotate the selected layer with touch
  gestures; tap to select, delete, bring-to-front; per-canvas aspect ratio and background; color +
  brush controls. (Stickers are emoji drawn as text layers, so they transform and export like any
  other layer.) The interactive preview draws through the same `FreeformRenderer` the exporter uses,
  so it stays pixel-faithful to the saved file.
- **Save & reopen projects** (`ui.projects` + `data.local`): tap the bookmark icon in any editor
  to save. Projects are stored as JSON plus a copy of their images in app-private storage — the copy
  matters because Photo Picker Uris are session-scoped and would otherwise stop working after a
  restart. The Recent Projects screen lists saved work with thumbnails; tap to reopen and keep
  editing, or delete. Only parameters + image copies are stored, never edited pixels.
- **Export** (all three): JPG / PNG / WEBP × Low / Medium / High / Maximum ×
  Original / 1080p / 1440p / 4K, plus **exact social export sizes** (IG post 1080×1350, Story 1080×1920, etc., cover-cropped to fit), saved to the gallery, with a **Share** action. A blocking
  progress overlay is shown while exporting.
- **Appearance**: Light / Dark / System theme toggle (Settings), persisted.
- **Ads** (through the seam): a **home banner** and a **frequency-capped interstitial after
  export** (every 3rd export), plus the **rewarded** 4K-export unlock. **Now wired to the real Google Mobile Ads SDK** (`play-services-ads` + UMP consent),
  using Google **test ad unit ids** during development (flip `AdIds.USE_TEST_ADS` for release).
- **Settings & monetization seam** (`ui.settings` + `ads`): the app is **offline and ad-free by
  default** (no INTERNET permission). Ads sit behind an `AdsController` interface with a no-op
  implementation, so 4K export is gated behind a **rewarded-ad unlock** that works end-to-end in
  development (the no-op simulates the ad and grants the reward, persisted via `Entitlements`).
  Settings offers the unlock, ad-privacy/consent management, and About. A real Google Mobile Ads
  (Next-Gen SDK) implementation drops in via `integration/AdMobAdsController.kt.txt` — no other
  code changes needed.
- **Error handling**: decode failures, out-of-memory, and export failures surface as snackbars.

---

## Roadmap (stubbed or not yet built)

**On-device AI:** the Freeform "Cutout" tool removes an image's background using **ML Kit
Subject Segmentation** — it runs on-device, but the model downloads once via Google Play
Services on first use (so the first cutout needs connectivity + Play Services; everything else
stays offline).

These are intentionally left as clear next steps:

- **More from the reference designs**: image-based pattern packs (the built-in patterns are drawn
  procedurally today).
- **More editor extras**: resize/scale to explicit dimensions, and highlights/shadows recovery.
  (Crop, straighten, blur, sharpen, and vignette are done; these remaining ones need either a
  resampling pass or tone-curve math beyond the current color matrix.)
- **Turn on real ads**: the monetization seam is built (interface, 4K rewarded gate, consent
  hook, Settings privacy controls). To go live, follow `integration/AdMobAdsController.kt.txt`:
  add the SDK + INTERNET permission + AdMob IDs and swap the controller in `AppContainer`.
  Natural next ad slots: a Home banner and a capped post-export interstitial.

---

## Tech

Kotlin 2.0 · Jetpack Compose (BOM 2024.09) · Material 3 · Navigation Compose · Coil ·
kotlinx-serialization · Coroutines · AndroidX ExifInterface · min SDK 24 / target 34.
