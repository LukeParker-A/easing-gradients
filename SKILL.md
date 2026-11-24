---
name: easing-gradients
description: Implement or tune gradients in UI code, including color fills, hero image fades, scroll-edge masks, and blends into a solid surface. Use by default when adding or changing a UI gradient. Choose the transition range separately from its easing, preserve the subject and layout, and use sampled eased stops for fades whose edges should disappear.
---

# Ease UI Gradients

## Choose the range and curve separately

1. Inspect the rendered transition and its source layers. Check whether the image already fades into a color or transparency, where the image is cropped, and which surface is visible at the end. An overlay cannot recover detail already faded in the asset.
2. Set the start and end independently. To lengthen a photo fade while preserving the subject, extend the visible source image at its lower edge when possible. Use existing space before the next section if the layout should stay fixed. Preserve image scale, the upper crop, and the next section's position. Use the actual bounds in each viewport or orientation.
3. For a fade into a surface, start with `ease-in-out` (`cubic-bezier(0.42, 0, 0.58, 1)`). The transition should start gently and flatten as it reaches the destination. If the subject gets washed out, inspect the midpoint opacity and the cropped source before choosing between moving the range and changing the curve. Tune against the actual image and surface.
4. Preserve explicit design stops, brand colors, and functional interpolation. Keep linear mixing for designs or data encodings that call for it. Apply easing to the part of a gradient that should fade gently into its surroundings.

## Render the fade

- For a surface-color overlay, interpolate the alpha of the actual adjacent surface from zero to one. Keep its RGB channels consistent, including at zero alpha. For a transparency mask, interpolate mask opacity from one to zero so the real background shows through.
- When the renderer accepts only linear color stops, sample the curve across the transition. Start with 16 equal intervals, including both endpoints. Keep the computed stops stable until the theme or gradient parameters change. Keep image and mask together when they zoom or move during a gesture.
- For CSS, emit ordinary `linear-gradient()` stops with explicit positions. For SVG, use `stop-opacity` with a consistent `stop-color`. The [Easing Gradients reference](https://larsenwork.com/easing-gradients/) illustrates how intermediate stops approximate the curve.

### Flutter example

This overlay fades through the lower 30% of a box. Set `surface` to the actual adjacent color. Cache the gradient for fixed inputs or rebuild it when those inputs change.

```dart
LinearGradient surfaceFade(Color surface) {
  const segments = 16;
  return LinearGradient(
    begin: const Alignment(0, 0.4),
    end: Alignment.bottomCenter,
    colors: List<Color>.generate(
      segments + 1,
      (index) => surface.withValues(
        alpha: Curves.easeInOut.transform(index / segments),
      ),
      growable: false,
    ),
  );
}
```

For `BoxFit.cover` when the width determines the photo scale, let `a = (alignmentY + 1) / 2` be the top fraction of the crop. To expose `extra` logical points below with the same composition, increase the image box height by `extra / (1 - a)` and shift it up by `a * extra / (1 - a)`, for `a < 1`. Clip it to the visible region. Check that the width still determines scale at the narrowest supported size. If the fit changes, compute the crop from the actual source and destination dimensions.

## Verify the result

Compare before and after at the same viewport, scale, theme, and scroll position. Check that the subject stays readable, the transition has no visible cutoff or white band, and the last pixels join the adjacent surface. For a resized or moving image, check the supported orientations and the gesture at rest, during motion, and after release. Verify any layout or interaction affected by the change.
