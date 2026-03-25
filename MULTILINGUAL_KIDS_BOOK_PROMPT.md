# 📖 HOW TO USE THIS PROMPT

1. Copy everything from **THE PROMPT** section below
2. Paste it to Claude
3. Claude will ask you which book — answer with title, author, language
4. Claude builds the full illustrated multilingual PDF

---

## THE PROMPT

---

Read `/home/claude/MULTILINGUAL_KIDS_BOOK_SKILL.md` first, then ask me the questions below one by one. After I answer all of them, build the complete illustrated multilingual children's book PDF using the exact pipeline from that skill file.

## Questions to ask me (in this order):

**1. Book details**
- What is the title of the book?
- Who is the author?
- What language is the original story in?
- How many chapters/scenes does it have? (recommend 6–8)

**2. Story content**
For each chapter, ask me:
- Chapter title (in source language)
- Which scene background? Choose: `forest | meadow | home | night | road | storm | winter | city`
- The story text in source language (3–5 short sentences, no fancy punctuation)
- Translation of that text into: English, Deutsch, Italiano, Français, Español

**3. Keywords**
For each chapter, ask for 6 important words with emoji and IPA pronunciation in all 5 languages.
Format: `emoji | word | /IPA/` — one per language per keyword slot.

**4. Characters and emoji map**
Ask for the main characters, objects and key words in the source language (all forms/cases) and their emoji.
Remind me that the map needs entries for ALL 6 languages (source + 5 translations).

**5. Output filename**
Ask: what should the PDF file be called? (no spaces, e.g. `crvenkapica.pdf`)

---

## THEN build the PDF

Once you have all the information:

### Technical requirements (DO NOT change)
- Page size: 16.5 × 8.5 inches at 120 DPI (1980 × 1020 px)
- All text rendered through PIL — never through ReportLab
- NotoColorEmoji loaded at size 109, downscaled with LANCZOS
- Story body: `mfont(19)`, emoji_sz=24, max_lines=6
- Translation body: `mfont(14)`, emoji_sz=18
- Language badge labels: `hfont(15)`, use_hand=True
- Keyword word: `mfont(12)`
- IPA pronunciation: **`ifont(12)` (DejaVuSans) — NEVER Poppins**
- render_clean_word MUST use `-x0, -y0` offset (not hardcoded `(1,2)`)
- render_hand_line MUST use grapheme_split() so flag emoji render correctly
- LANG_N = 5 matching exactly 5 entries in LANG_ROWS
- Source language NOT in LANG_ROWS
- WORD_EMOJI must include entries for all 6 languages (source + EN/DE/IT/FR/ES)
- Each chapter has unique wobble seeds (seed=chapter_idx*10+offset)
- Translation table: LEFT col = text (56%), RIGHT col = 3×2 keyword grid (44%)
- Keyword cells: emoji + word (mfont 12) + /pron/ (ifont 12, slightly lighter ink)

### Output
Save to: `/mnt/user-data/outputs/[filename].pdf`

---

## EXAMPLE — filled in for Crvenkapica (Little Red Riding Hood)

```
Title: Crvenkapica
Author: Braća Grimm
Source language: Croatian
Chapters: 6
Output: crvenkapica.pdf

Chapter 1 — "Crvenkapica" | scene: home
HR: Bila je jednom djevojčica s crvenom kapom. Svi su je zvali Crvenkapica. Živjela je s mamom na rubu šume.
EN: Once there was a little girl with a red hood. Everyone called her Little Red Riding Hood. She lived with her mother at the edge of the forest.
DE: Es war einmal ein Mädchen mit einer roten Kappe. Alle nannten sie Rotkäppchen. Sie lebte mit ihrer Mutter am Waldrand.
IT: C'era una volta una bambina con un cappuccio rosso. Tutti la chiamavano Cappuccetto Rosso. Viveva con sua mamma ai margini del bosco.
FR: Il était une fois une petite fille avec un chaperon rouge. Tout le monde l'appelait le Petit Chaperon Rouge. Elle vivait avec sa maman à la lisière de la forêt.
ES: Había una vez una niña con una capa roja. Todos la llamaban Caperucita Roja. Vivía con su mamá al borde del bosque.

Keywords chapter 1:
EN: 🧒 girl /ɡɜːl/ | 🔴 red /rɛd/ | 🏠 home /hoʊm/ | 🌲 forest /ˈfɒrɪst/ | 👩 mother /ˈmʌðər/ | 🧢 hood /hʊd/
DE: 🧒 Mädchen /ˈmɛːtçən/ | 🔴 rot /roːt/ | 🏠 Zuhause /tsuˈhaʊzə/ | 🌲 Wald /valt/ | 👩 Mutter /ˈmʊtər/ | 🧢 Kappe /ˈkapə/
IT: 🧒 bambina /bamˈbiːna/ | 🔴 rosso /ˈrɔsso/ | 🏠 casa /ˈkaːza/ | 🌲 bosco /ˈbɔsko/ | 👩 mamma /ˈmamma/ | 🧢 cappuccio /kapˈputtʃo/
FR: 🧒 fille /fij/ | 🔴 rouge /ʁuʒ/ | 🏠 maison /mɛˈzɔ̃/ | 🌲 forêt /fɔˈʁɛ/ | 👩 maman /maˈmɑ̃/ | 🧢 chaperon /ʃapˈʁɔ̃/
ES: 🧒 niña /ˈniɲa/ | 🔴 roja /ˈroxa/ | 🏠 casa /ˈkasa/ | 🌲 bosque /ˈboske/ | 👩 mamá /maˈma/ | 🧢 capa /ˈkapa/

Emoji map (source language):
djevojčica, djevojčice → 🧒
crvena, crvenoj → 🔴
kapom, kapa, kapu → 🧢
šuma, šume, šumi → 🌲
mama, mame → 👩
vuk, vuka, vuku → 🐺
baka, bake, baki → 👵
kolač, kolača → 🎂
```

---

## TIPS

**Short sentences** — 3–5 per chapter. The story text zone is ~310px tall.

**Scene tags** — pick the one that matches the chapter mood best:
- `forest` → trees, dark canopy, bear+fox figures
- `meadow` → hills, flowers, bright sky
- `home` → house, green garden, blue sky
- `night` → stars, moon, dark sky
- `road` → dirt path, trees, sky
- `storm` → dark clouds, rain
- `winter` → snow, pale sky, snowflakes
- `city` → buildings with lit windows

**IPA tips** — use standard IPA. For tricky languages:
- French nasals: ɑ̃ ɛ̃ ɔ̃ õ
- Spanish: β ð ɣ ɾ θ
- German: ç ʏ œ ʊ
- Italian: tʃ dʒ ɔ ɛ

**Adding a new scene** — add `elif scene=="tag":` branches in both `draw_scene_bg()` and `make_mini_scene()`.

**More chapters** — ~3–4 seconds per page. 8 chapters ≈ 30 seconds.
