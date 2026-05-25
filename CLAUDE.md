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
