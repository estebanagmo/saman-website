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

recolor("assets/images/logo-saman.png", "#17181c")
```

Run with: `py recolor_logo.py` (requires Pillow: `pip install pillow`)

Note: `logo-saman-white.png` is the same shape in pure white, for dark backgrounds
(the footer) — leave it as is.

## Color palette

Both pages share the app's design system ("SaaS Pulido" —
`C:\Users\EstebanAguel\saman\design\design-system.md`). Tokens live in the `:root` of
`index.html` (inline) and `assets/style.css`. Keep the two in sync.

| Token | Hex | Role |
|---|---|---|
| `--action` | `#4f46e5` | Purple — action controls (buttons, active language toggle) |
| `--action-hover` | `#4338ca` | Action hover |
| `--text` | `#17181c` | Headings and the logo |
| `--ink` / `--navy` | `#4b4e58` | Slate — dark fills (footer, contact hero). Never pure black. |
| `--muted` | `#6b6f7b` | Body copy |
| `--faint` | `#9a9eab` | Labels, timestamps |
| `--border` | `#e8e9ee` | Dividers |
| `--border2` | `#dcdee6` | Control borders (inputs, ghost buttons) |
| `--surface` | `#f7f8fa` | Page background |
| `--sel-bg` / `--blue-light` | `#f1f2f6` | Neutral selection/emphasis |

The app reserves purple strictly for action controls, but the homepage headline
deliberately breaks that rule: "automatiza" is `--action` (explicit request 2026-08-11).

Bump the `?v=` cache buster on the `style.css` link in `contact.html` whenever the
stylesheet changes.
