# Saman Website

## Utilities

### Remove white background from images (`remove_bg.py`)

Used to strip white/near-white backgrounds from PNG images so corners are transparent. Run this whenever new VP section images are added that have opaque white backgrounds (e.g. exported from Claude Design).

```python
from PIL import Image
import numpy as np
from collections import deque

def remove_background_flood_fill(path, threshold=240):
    img = Image.open(path).convert("RGBA")
    data = np.array(img)
    h, w = data.shape[:2]

    is_near_white = (
        (data[:,:,0] > threshold) &
        (data[:,:,1] > threshold) &
        (data[:,:,2] > threshold)
    )

    background = np.zeros((h, w), dtype=bool)
    queue = deque()

    # Seed BFS from all four edges
    for x in range(w):
        for y in [0, h - 1]:
            if is_near_white[y, x] and not background[y, x]:
                background[y, x] = True
                queue.append((y, x))
    for y in range(h):
        for x in [0, w - 1]:
            if is_near_white[y, x] and not background[y, x]:
                background[y, x] = True
                queue.append((y, x))

    # Flood fill outward through connected near-white pixels only
    while queue:
        y, x = queue.popleft()
        for dy, dx in [(-1,0),(1,0),(0,-1),(0,1)]:
            ny, nx = y + dy, x + dx
            if 0 <= ny < h and 0 <= nx < w and not background[ny, nx] and is_near_white[ny, nx]:
                background[ny, nx] = True
                queue.append((ny, nx))

    data[background, 3] = 0
    Image.fromarray(data).save(path, "PNG")
    print(f"Processed: {path}")

images = [
    "assets/images/vp-visibilidad.png",
    "assets/images/vp-financiacion.png",
    "assets/images/vp-tiempo.png",
]

for img_path in images:
    remove_background_flood_fill(img_path)
```

Run with: `py remove_bg.py` (requires Pillow: `pip install pillow`)

### Round image corners (`round_corners.py`)

Used to clip the corners of a PNG to a rounded rectangle, matching the macOS browser window corner curvature. Use this on browser screenshot images that show dark/white corner artifacts from the rectangular PNG bounding box.

```python
from PIL import Image, ImageDraw

def apply_rounded_corners(path, radius=20):
    img = Image.open(path).convert("RGBA")
    w, h = img.size

    mask = Image.new("L", (w, h), 0)
    draw = ImageDraw.Draw(mask)
    draw.rounded_rectangle([(0, 0), (w - 1, h - 1)], radius=radius, fill=255)

    img.putalpha(mask)
    img.save(path, "PNG")
    print(f"Done: {path} — radius={radius}px on {w}x{h}")

apply_rounded_corners("assets/images/vp-visibilidad.png", radius=20)
```

Run with: `py round_corners.py` (requires Pillow: `pip install pillow`)

Note: `radius=20` works for 2880×1800 retina screenshots. Adjust if using a different resolution.

### Recolor the wordmark (`recolor_logo.py`)

`logo-saman.png` is a flat single-color wordmark. This swaps its color while keeping the
antialiased letter edges intact (it rewrites RGB and leaves the alpha channel alone).
Used to move the logo off the old green onto the shared palette.

```python
from PIL import Image
import numpy as np

def recolor(path, hex_color, out=None):
    r, g, b = (int(hex_color.lstrip("#")[i:i+2], 16) for i in (0, 2, 4))
    img = Image.open(path).convert("RGBA")
    data = np.array(img)

    # Replace RGB everywhere, keep alpha so the antialiased edges survive.
    data[:, :, 0] = r
    data[:, :, 1] = g
    data[:, :, 2] = b

    Image.fromarray(data).save(out or path, "PNG")
    print(f"Recolored {out or path} -> {hex_color}")

recolor("assets/images/logo-saman.png", "#262451")
```

Run with: `py recolor_logo.py` (requires Pillow: `pip install pillow`)

Note: since the 2026-10 "Tinta índigo" palette every footer is light and uses
`logo-saman.png`. `logo-saman-white.png` (pure white) is no longer referenced but is kept
for any future dark background.

## Color palette

"Tinta índigo" (2026-10): the app's design system ("SaaS Pulido" —
`C:\Users\EstebanAguel\saman\design\design-system.md`) with the black and slate moved to a
very dark indigo. Tokens live in the `:root` of `assets/style.css` (used by `index.html`,
`encuesta2026.html`, `encuesta2026-datos.html`, `contact.html`, `404.html`) and inline in
the legal pages (`privacidad`, `privacy`, `terminos`, `terms`). Keep them in sync. The
names differ: style.css calls the purple `--indigo`/`--indigo-dark`, the legal pages
`--action`/`--action-hover`.

| Token | Hex | Role |
|---|---|---|
| `--indigo` / `--action` | `#4f46e5` | Purple — buttons, the Beta chip, highlighted phrases |
| `--indigo-dark` / `--action-hover` | `#4338ca` | Action hover |
| `--text` | `#262451` | Headings and the logo (indigo ink, not black) |
| `--navy` | `#262451` | Dark fills (contact hero, step numbers). Never black or slate. |
| `--body` / `--ink` | `#4d5170` | Body text |
| `--muted` | `#6a6e88` | Secondary copy, footer text |
| `--border` | `#e9e9f3` | Dividers |
| `--border2` | `#dcdcec` | Control borders (inputs, ghost buttons) |
| `--surface` | `#f7f7fc` | Lavender-tinted page background |
| `--sel-bg` / `--blue-light` | `#f0f0f8` | Neutral selection/emphasis |

Fixed values outside the tokens: footer `#f3f3fa`, closing CTA band `#eeedfd`, "Disponible" pill border and text-link underline `#c7c4f6`.

Bump the `?v=` cache buster on every `style.css` link (index, encuesta2026,
encuesta2026-datos, contact, 404) whenever the stylesheet changes.

## Typography and logo

Body text is Inter. Headings (`h1`–`h3`, `.founder__name`) and the logo use Plus Jakarta Sans
through the `--brand` token in `style.css`; only `index`, `encuesta2026` and
`encuesta2026-datos` load it from Google Fonts, so contact, 404 and the legal pages still
fall back to Inter. On those three pages the logo is typed, not an image:
`<a class="nav__logo wordmark"><img src="assets/images/logo-arbol.png" alt="">Samán</a>`
(the tree, cropped from `logo-saman.png`, plus live text at 700, −0.035em). The other pages
still use the old `logo-saman.png` wordmark.

## Hosting

GitHub Pages (`CNAME`), no build step. `/foo` resolves to `foo.html`; `/foo/` does not,
so `404.html` redirects `/encuesta2026/` to `/encuesta2026`, keeping `?src=`.
Placeholders still to fill before launch are listed in `CHANGES.md`.
