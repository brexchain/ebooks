# 📚 SKILL: Illustrated Multilingual Kids Book PDF
## Version 2 — battle-tested, all bugs fixed

---

## What this skill produces

A landscape PDF (16.5 × 8.5 in, 120 DPI) — **one page per chapter**:

```
┌──────────────────────────────────────────────────────────────────┐
│  ✦  Chapter number  •  TITLE BANNER (wobbly coloured strip)  ✦  │
├────────────────────────────────────────────────────────────────── │
│  [mini scene art]  │  Story text (source language)               │
│   bear + fox       │  every keyword → inline emoji 🐻            │
├── ⭐ ⭐ ⭐  star divider  ⭐ ⭐ ⭐ ─────────────────────────────│
│  [🇬🇧 ENGLISH badge] │ translation text+emoji │ 3×2 keyword grid │
│  [🇩🇪 DEUTSCH badge] │ translation text+emoji │ 3×2 keyword grid │
│  [🇮🇹 ITALIANO badge]│ translation text+emoji │ 3×2 keyword grid │
│  [🇫🇷 FRANÇAIS badge]│ translation text+emoji │ 3×2 keyword grid │
│  [🇪🇸 ESPAÑOL badge] │ translation text+emoji │ 3×2 keyword grid │
└──────────────────────────────────────────────────────────────────┘
```

Each keyword cell in the 3×2 grid shows:
```
[emoji]  word
         /IPA pronunciation/
```

---

## Environment

```bash
pip install reportlab pillow --break-system-packages
```

### Required fonts (pre-installed on system)
| Variable   | Path                                                               | Purpose                        |
|------------|--------------------------------------------------------------------|--------------------------------|
| EMOJI_PATH | `/usr/share/fonts/truetype/noto/NotoColorEmoji.ttf`               | Full-colour emoji (load at 109)|
| BOLD_PATH  | `/usr/share/fonts/truetype/google-fonts/Poppins-Bold.ttf`         | Titles, banners                |
| MEDIUM_PATH| `/usr/share/fonts/truetype/google-fonts/Poppins-Medium.ttf`       | Body text                      |
| HAND_PATH  | `/usr/share/fonts/truetype/google-fonts/Poppins-BoldItalic.ttf`   | Badge labels (wobbly)          |
| REG_PATH   | `/usr/share/fonts/truetype/google-fonts/Poppins-Regular.ttf`      | Author, subtitles              |
| IPA_PATH   | `/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf`                 | **IPA pronunciation ONLY**     |

> ⚠️ **CRITICAL**: Poppins does NOT support IPA characters (ɛ ɔ ʁ ˈ ɥ β ð ɾ θ ʃ …).
> Always use `ifont()` / DejaVuSans for pronunciation text. Using `rfont()` (Poppins) causes black squares.

---

## Page constants — copy verbatim

```python
W_IN, H_IN = 16.5, 8.5
DPI = 120
W, H = int(W_IN * DPI), int(H_IN * DPI)   # 1980 × 1020

BANNER_H = 62        # top chapter banner height
STORY_H  = 310       # story zone height (below banner)
TABLE_Y  = BANNER_H + STORY_H + 10
TABLE_H  = H - TABLE_Y - 12
LANG_N   = 5         # number of translation languages
```

---

## Font loaders — copy verbatim

```python
_emoji_native = ImageFont.truetype(EMOJI_PATH, 109)  # ALWAYS load at 109

def bfont(s): return ImageFont.truetype(BOLD_PATH,   s)
def mfont(s): return ImageFont.truetype(MEDIUM_PATH, s)   # body text
def hfont(s): return ImageFont.truetype(HAND_PATH,   s)   # badge labels only
def rfont(s): return ImageFont.truetype(REG_PATH,    s)   # subtitles
def ifont(s): return ImageFont.truetype(IPA_PATH,    s)   # IPA ONLY — DejaVu
```

---

## Critical rendering functions — copy verbatim

### render_emoji_img
```python
def render_emoji_img(ch, size):
    tmp = Image.new("RGBA", (128, 128), (0,0,0,0))
    ImageDraw.Draw(tmp).text((0,0), ch, font=_emoji_native, embedded_color=True)
    bb = tmp.getbbox()
    if bb: tmp = tmp.crop(bb)
    return tmp.resize((size, size), Image.LANCZOS)
```
> Load NotoColorEmoji at 109, downscale via LANCZOS. Never pass emoji to ReportLab.

### render_clean_word  ← fixes IPA baseline clipping
```python
def render_clean_word(text, font, ink):
    dummy = Image.new("RGBA", (1,1))
    bb = ImageDraw.Draw(dummy).textbbox((0,0), text, font=font)
    x0, y0 = bb[0], bb[1]           # MUST account for offset — IPA chars have unusual baselines
    tw = max(1, bb[2] - x0 + 4)
    th = max(1, bb[3] - y0 + 6)
    img = Image.new("RGBA", (tw, th), (0,0,0,0))
    ImageDraw.Draw(img).text((-x0 + 1, -y0 + 2), text, font=font, fill=ink)
    return img
```
> **Never** use `(1, 2)` as fixed draw origin — always offset by `-x0, -y0`. IPA fails otherwise.

### grapheme_split + render_hand_line  ← fixes flag emoji (🇬🇧 etc.)
```python
def grapheme_split(text):
    """Keeps flag emoji (2 regional indicator codepoints) together as one token."""
    REGIONAL = range(0x1F1E6, 0x1F1E6 + 26)
    chars = list(text)
    clusters = []
    i = 0
    while i < len(chars):
        if i+1 < len(chars) and ord(chars[i]) in REGIONAL and ord(chars[i+1]) in REGIONAL:
            clusters.append(chars[i] + chars[i+1])
            i += 2
        else:
            clusters.append(chars[i])
            i += 1
    return clusters

def render_hand_line(text, font_size, ink, seed=0, use_hand=False):
    font = hfont(font_size) if use_hand else mfont(font_size)
    if not use_hand:
        return render_clean_word(text, font, ink)
    rng = random.Random(seed)
    clusters = grapheme_split(text)
    imgs = []
    for cl in clusters:
        if len(cl) == 2 and ord(cl[0]) in range(0x1F1E6, 0x1F1E6 + 26):
            imgs.append(render_emoji_img(cl, font_size + 4))  # flag via emoji engine
        else:
            imgs.append(render_hand_char(cl, font, ink, rng))
    total_w = sum(i.width for i in imgs) + 2*(len(imgs)-1)
    total_h = max(i.height for i in imgs) + 8
    out = Image.new("RGBA", (max(1,total_w), max(1,total_h)), (0,0,0,0))
    x = 0
    for im in imgs:
        out.paste(im, (x, (total_h - im.height)//2), im)
        x += im.width + 2
    return out
```
> Flag emoji (🇬🇧 🇩🇪 🇮🇹 🇫🇷 🇪🇸) are TWO Unicode codepoints each.
> Splitting char-by-char renders each half as a black square. Always use grapheme_split.

### render_hand_char  (used internally by render_hand_line)
```python
def render_hand_char(ch, font, ink, rng):
    dummy = Image.new("RGBA", (1,1))
    bb = ImageDraw.Draw(dummy).textbbox((0,0), ch, font=font)
    tw, th = max(1, bb[2]-bb[0]+4), max(1, bb[3]-bb[1]+8)
    tmp = Image.new("RGBA", (tw+4, th+4), (0,0,0,0))
    jitter_y = rng.randint(-1, 1)
    angle = rng.uniform(-2.5, 2.5)
    ImageDraw.Draw(tmp).text((2, 2+jitter_y), ch, font=font, fill=ink)
    return tmp.rotate(angle, expand=False, resample=Image.BICUBIC)
```

### render_rich_block  (word-wrap + inline emoji)
```python
def render_rich_block(text, max_w, font_size, ink, emoji_sz,
                      seed=0, max_lines=None, bold=False):
    font = bfont(font_size) if bold else mfont(font_size)
    words = text.split()
    line_h = font_size + emoji_sz + 3
    measured = []
    for w in words:
        wi = render_clean_word(w + " ", font, ink)
        em = get_emoji(w)
        em_img = render_emoji_img(em, emoji_sz) if em else None
        em_w = emoji_sz + 2 if em_img else 0
        measured.append((w, wi, em_img, em_w))
    lines = []
    cur, cur_w = [], 0
    for w, wi, em_img, em_w in measured:
        need = wi.width + em_w
        if cur and cur_w + need > max_w:
            lines.append(cur)
            cur, cur_w = [], 0
        cur.append((w, wi, em_img, em_w))
        cur_w += need
    if cur: lines.append(cur)
    if max_lines: lines = lines[:max_lines]
    total_h = len(lines) * line_h
    out = Image.new("RGBA", (max_w, max(1, total_h)), (0,0,0,0))
    y = 0
    for line in lines:
        x = 0
        for w, wi, em_img, em_w in line:
            out.paste(wi, (x, y + (line_h - wi.height)//2), wi)
            x += wi.width
            if em_img:
                out.paste(em_img, (x, y + (line_h - emoji_sz)//2), em_img)
                x += em_w
        y += line_h
    return out
```

### wobble_rect
```python
def wobble_rect(draw, x, y, w, h, color, fill=None, seed=0, lw=4, steps=90):
    rng = random.Random(seed)
    pts = []
    for i in range(steps//4):
        t = i/(steps//4)
        pts.append((x + t*w + rng.uniform(-4,4), y + rng.uniform(-4,4)))
    for i in range(steps//4):
        t = i/(steps//4)
        pts.append((x + w + rng.uniform(-4,4), y + t*h + rng.uniform(-4,4)))
    for i in range(steps//4):
        t = i/(steps//4)
        pts.append((x + w - t*w + rng.uniform(-4,4), y + h + rng.uniform(-4,4)))
    for i in range(steps//4):
        t = i/(steps//4)
        pts.append((x + rng.uniform(-4,4), y + h - t*h + rng.uniform(-4,4)))
    if fill: draw.polygon(pts, fill=fill)
    draw.line(pts + [pts[0]], fill=color, width=lw)
```

---

## Font size reference (proven readable in final PDF)

| Element                  | Font        | Size | Notes                              |
|--------------------------|-------------|------|------------------------------------|
| Chapter title (banner)   | hfont bold  | 24   | use_hand=True                      |
| Chapter number (circle)  | hfont bold  | 22   | use_hand=True                      |
| Story body text          | mfont       | 19   | emoji_sz=24, max_lines=6           |
| Translation body text    | mfont       | 14   | emoji_sz=18, max_lines per row_h   |
| Language badge label     | hfont       | 15   | use_hand=True, includes flag emoji |
| Keyword word label       | mfont       | 12   | in 3×2 grid                        |
| IPA pronunciation        | **ifont**   | **12** | **DejaVu ONLY — never Poppins**  |
| Cover title              | bfont       | 72   |                                    |
| Cover author             | rfont       | 38   |                                    |

---

## Data structures

### CHAPTERS list
```python
CHAPTERS = [
    (
        "Title in source language",       # str
        "scene_tag",                      # see scene table below
        "Body text. 3-5 short sentences. No fancy punctuation.",
        {
            "🇬🇧 ENGLISH":  "Translation...",
            "🇩🇪 DEUTSCH":  "Übersetzung...",
            "🇮🇹 ITALIANO": "Traduzione...",
            "🇫🇷 FRANÇAIS": "Traduction...",
            "🇪🇸 ESPAÑOL":  "Traducción...",
        }
    ),
    ...
]
```

### CHAPTER_KEYWORDS list  ← one dict per chapter, same order as CHAPTERS
```python
CHAPTER_KEYWORDS = [
    {   # Chapter 1
        "🇬🇧 ENGLISH":  [("🐻","bear","bɛr"), ("🦊","fox","fɒks"), ...],   # 6 tuples
        "🇩🇪 DEUTSCH":  [("🐻","Bär","bɛːr"), ("🦊","Fuchs","fʊks"), ...],
        "🇮🇹 ITALIANO": [...],
        "🇫🇷 FRANÇAIS": [...],
        "🇪🇸 ESPAÑOL":  [...],
    },
    ...  # one dict per chapter
]
```
> Each list has exactly **6 tuples**: `(emoji, word_in_that_language, IPA_pronunciation)`.
> Grid is 3 columns × 2 rows. IPA must be accurate for the language.

### WORD_EMOJI dict
Map every important word **in ALL languages** (source + 5 translations) to an emoji.
Include all inflected/conjugated forms. Minimum ~80 entries.

```python
WORD_EMOJI = {
    # Source language forms
    "wolf": "🐺", "wolves": "🐺", "wolfs": "🐺",
    # English
    "wolf": "🐺", "howl": "🌕",
    # German
    "wolf": "🐺", "wölfe": "🐺",
    # Italian
    "lupo": "🐺", "lupi": "🐺",
    # French
    "loup": "🐺", "loups": "🐺",
    # Spanish
    "lobo": "🐺", "lobos": "🐺",
}

def get_emoji(word):
    return WORD_EMOJI.get(re.sub(r"[^\w]", "", word, flags=re.UNICODE).lower())
```

### Scene tags
| Tag      | Background drawn                          |
|----------|-------------------------------------------|
| `forest` | Dark trees, layered canopy                |
| `meadow` | Sky gradient, rolling hills, flowers      |
| `home`   | Sky, green ground, house with roof        |
| `night`  | Star field, crescent moon, dark ground    |
| `road`   | Sky, dirt road with dashes, trees         |
| `storm`  | Dark sky, rain streaks, ground            |
| `winter` | Pale sky, snow ground, snowflakes         |
| `city`   | Buildings with lit windows, sky           |
| `fire`   | Orange glow sky (use city fallback)       |

Add new tags by adding `elif scene=="tag":` in both `draw_scene_bg()` and `make_mini_scene()`.

### Colour palettes  (one per chapter, cycles)
```python
PALETTES = [
    {"bg":(R,G,B), "border":(R,G,B), "banner":(R,G,B), "title":(R,G,B)},
    ...
]
```
Rules: `bg` = pastel (200-255 per channel), `border` = vivid, `banner` between both, `title` = darkest.

### Language rows
```python
LANG_ROWS = [
    ("🇬🇧 ENGLISH",  ink_rgb, row_bg_rgb, badge_bg_rgb),
    ("🇩🇪 DEUTSCH",  ...),
    ("🇮🇹 ITALIANO", ...),
    ("🇫🇷 FRANÇAIS", ...),
    ("🇪🇸 ESPAÑOL",  ...),
]
```
> Source language is NOT in LANG_ROWS — it's already in the story zone above.

---

## Translation table layout (per row)

```
[accent bar 8px] [wobbly badge: flag+name] | [LEFT col: translated text+emoji] | [RIGHT col: 3×2 keyword grid]
```

Column widths:
- Badge area: ~175px fixed
- LEFT column: 56% of remaining width
- RIGHT column: 44% of remaining width (for 3 keyword cells × 2 rows)

Keyword cell rendering:
```python
# emoji at left, word + /pron/ stacked at right
em_sz = min(18, KW_CELL_H - 4)
em_img = render_emoji_img(em, em_sz)
word_img = render_clean_word(word, mfont(12), ink)
light_ink = tuple(min(200, c + 40) for c in ink)   # slightly lighter, still readable
pron_img  = render_clean_word(f"/{pron}/", ifont(12), light_ink)  # ifont = DejaVu!
```

---

## PDF assembly
```python
def build_pdf(out_path):
    os.makedirs(os.path.dirname(out_path), exist_ok=True)
    pdf_w, pdf_h = W_IN * 72, H_IN * 72
    c = canvas.Canvas(out_path, pagesize=(pdf_w, pdf_h))
    pages = [make_cover()] + [make_page(i, *ch, PALETTES[i % len(PALETTES)]) for i, ch in enumerate(CHAPTERS)]
    for pg in pages:
        buf = io.BytesIO()
        pg.save(buf, format="JPEG", quality=90)
        buf.seek(0)
        c.drawImage(ImageReader(buf), 0, 0, pdf_w, pdf_h)
        c.showPage()
    c.save()
```

---

## Common mistakes — all battle-tested fixes included

| Bug                              | Wrong                          | Correct                                      |
|----------------------------------|--------------------------------|----------------------------------------------|
| IPA shows as black squares       | `rfont()` / Poppins for pron   | `ifont()` / DejaVu for ALL pronunciation     |
| Flag emoji (🇬🇧) shows as squares | `list(text)` splits to 2 chars | `grapheme_split()` keeps regional pairs      |
| IPA text clipped/invisible       | `draw.text((1,2), ...)`        | `draw.text((-x0+1, -y0+2), ...)` in render_clean_word |
| No emojis in translated text     | WORD_EMOJI has source lang only| Add ALL 5 languages' vocabulary to WORD_EMOJI|
| Same border on all chapters      | Same seed for wobble_rect      | Unique seed per element: `seed=ch_idx*10+n`  |
| Source language in table         | Adding it to LANG_ROWS         | Source language is in story zone — never repeat|
| Font size too small to read      | pron at 9-10px                 | word=12px mfont, pron=12px ifont             |
