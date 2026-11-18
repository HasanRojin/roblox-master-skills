---
name: model-reference-sheet
description: >
  Turn a short description of any 3D game model (prop, item, weapon, pet, furniture, vehicle, character accessory)
  into a complete "model reference sheet" image prompt: hero render, orthographic views with a scale bar,
  part callouts, close-ups and a material palette with hex codes, all on one 16:9 sheet. Use this whenever the
  user asks for a model sheet, reference sheet, turnaround, orthographic views, concept sheet, asset spec image,
  "image for 3D modeling", or says something like "make the image for this model" or "write the prompt for this
  object", in English or Bangla (e.g. "মডেলের ইমেজ বানাও", "রেফারেন্স শিট", "এই মডেলের প্রম্পট দাও"), even if they
  do not say "reference sheet". Also use it before modeling an asset in Blender or Roblox Studio when no reference exists.
---

# Model Reference Sheet

Produce one image prompt (and the image itself, if an image-generation tool is available) for a professional
3D **model reference sheet**. A modeler, a Blender `bpy` script or an image-to-3D tool should be able to build the
asset from this sheet alone, so every panel exists to remove guesswork: shape from four sides, real size, named
parts, surface detail and exact colors.

## 1. Gather the brief (ask only what is missing)

From the user's words, fill in this brief. Infer sensible defaults and state them instead of asking a long list
of questions; ask at most one short question if the object itself is unclear.

| Field | Example | Default if not given |
| :-- | :-- | :-- |
| Object | small adventurer's pouch | (required) |
| Style | stylized chunky cartoon 3D, soft rounded bevels | the default style in Section 3 |
| Real height | 0.25 m | estimate a believable size for the object |
| Parts (3–5) | front flap, brass button, pouch body, stitched side patch | derive from the object |
| Colors (4) | flap purple, pouch purple, brass gold, stitch yellow | build a palette per Section 4 |
| Hero environment | cozy medieval tavern storeroom, candles, barrels | a themed scene that fits the game |
| Game / engine | Roblox | Roblox (add stud conversion) |

Why only 3–5 parts and 4 colors: image models garble crowded labels. A small, clear set reads correctly and
is enough for a modeler.

## 2. The sheet layout (fixed)

Always use this layout. It matches the user's established reference sheets, so all assets in the project look
like one consistent set.

```
+-----------------------------+------------------------------------------+
|                             |  ORTHOGRAPHIC VIEWS                      |
|  HERO RENDER                |  [scale bar "0.25 m"]                    |
|  3/4 view, in a themed      |   FRONT    RIGHT    BACK    TOP          |
|  environment, warm lighting |  (same scale, on one ground line)        |
+-------------+---------------+---------------------------+--------------+
| PART        | CLOSE-UPS                                 | MATERIALS    |
| CALLOUTS    | 2 rows x 3 tiles, each with a caption     | 4 swatches:  |
| 3/4 render, |                                           | name, #HEX,  |
| 4 leader    |                                           | finish       |
| lines       |                                           |              |
+-------------+-------------------------------------------+--------------+
```

- **Canvas:** 16:9 landscape (e.g. 1920×1080 or larger). Off-white warm background (#EFE9E3-ish) with
  rounded-corner panels and small gutters.
- **Headings:** bold, uppercase, dark brown (#3B2A20-ish), rounded sans-serif, top-left of each panel.
- **Hero render (top-left, ~45% width):** the object large, 3/4 view, inside a softly blurred themed
  environment that echoes the palette.
- **Orthographic views (top-right):** FRONT, RIGHT, BACK, TOP (or the four most informative views, e.g.
  FRONT (6), RIGHT (3) for a die). Same scale, flat even lighting, sitting on one thin ground line, labels
  centered below. A vertical dimension bar with end ticks on the left, labelled with the real height.
- **Part callouts (bottom-left):** a clean 3/4 render on a plain background, 4 thin black leader lines ending
  in small dots, with short two-line labels on the left (e.g. "Brass / Button", "Glowing Dot / (Inset)").
- **Close-ups (bottom-center):** a 2×3 grid of macro crops, each with a short caption below: key part details,
  the base surface texture, a structural detail (bevel, seam, hinge, belt loop), and one combined angle.
- **Materials (bottom-right):** 4 rows, each a rounded color swatch + name + hex code + finish
  (Matte / Satin / Glossy / Metallic / Emissive).

## 3. Default visual style

Stylized, chunky, toy-like 3D render: soft rounded bevels on every edge, slightly exaggerated proportions,
clean readable silhouettes, subtle fabric/plastic micro-texture, soft global illumination, gentle ambient
occlusion, no harsh specular. This style fits Roblox and casual games. If the user names another style
(low-poly, realistic PBR, voxel, hand-painted), swap only this paragraph and keep the layout.

## 4. Palette rules

- Exactly 4 materials: a primary, a secondary (a shade of the primary or a complement), an accent
  (metal/trim) and a detail color (stitching, glow, symbol).
- Give each a short name and a hex code, and use the **same hex codes** in the prompt text and in the
  Materials panel so the image and the spec agree.
- Mark self-lit parts as **Emissive**; they map to `Neon` material or `SurfaceAppearance` emissive in Roblox.
- Keep enough contrast that parts stay distinct in the orthographic views.

## 5. Output format (always this order)

1. **Brief** — the filled table from Section 1 (one line per field), noting which values you assumed.
2. **Image prompt** — one block in English, ready to paste into any image generator (Section 6 template).
3. **Negative prompt** — one line.
4. **Build spec** — real size, Roblox size in studs (1 stud ≈ 0.28 m, so studs = meters ÷ 0.28),
   suggested triangle budget (props ≤ 5,000), and the part list. This lets the next step (Blender `bpy`
   or Studio) start immediately; see `ai-prompts/image-and-3d-asset-prompts.md` in this repo.
5. If you have an image-generation tool, generate the image from the prompt, then check it against the
   checklist in Section 7 and say what came out wrong (text is the usual problem).

If the user writes in Bangla, write the explanation in Bangla but keep the image prompt in English, because
image models follow English prompts far more reliably.

## 6. Prompt template

Fill every `[...]`. Keep label words exactly as they should appear in the image, in quotes.

```text
A professional 3D game asset reference sheet for a [STYLE] [OBJECT], 16:9 landscape layout on a warm
off-white background, divided into rounded-corner panels with bold uppercase dark-brown headings.

Top-left panel: large hero render of the [OBJECT] in 3/4 view inside a softly blurred [ENVIRONMENT],
warm cinematic lighting, colors echoing the palette.

Top-right panel titled "ORTHOGRAPHIC VIEWS": four views of the same [OBJECT] side by side at identical
scale on one thin ground line, labelled "[VIEW 1]", "[VIEW 2]", "[VIEW 3]", "[VIEW 4]", flat even studio
lighting; on the left a vertical dimension bar with end ticks labelled "[HEIGHT] m".

Bottom-left panel titled "PART CALLOUTS": clean 3/4 render on a plain light background with four thin black
leader lines ending in small dots, labelled "[PART 1]", "[PART 2]", "[PART 3]", "[PART 4]".

Bottom-center panel titled "CLOSE-UPS": a grid of six square macro crops in two rows, captioned
"[CLOSE-UP 1]", "[CLOSE-UP 2]", "[CLOSE-UP 3]", "[CLOSE-UP 4]", "[CLOSE-UP 5]", "[CLOSE-UP 6]".

Bottom-right panel titled "MATERIALS": four rows, each with a rounded color swatch, a name, a hex code and a
finish: "[NAME 1] [#HEX1] [FINISH1]", "[NAME 2] [#HEX2] [FINISH2]", "[NAME 3] [#HEX3] [FINISH3]",
"[NAME 4] [#HEX4] [FINISH4]".

Object details: [2–4 sentences describing shape, proportions, each part, surface texture and which colors
go where, using the hex codes above].

Style: stylized chunky toy-like 3D render, soft rounded bevels, subtle micro-texture, soft global
illumination, gentle ambient occlusion, crisp clean typography, consistent object design across every panel.
```

**Negative prompt:**
```text
inconsistent object between panels, extra or missing parts, misspelled labels, garbled text, watermark, logo,
signature, photorealistic people, cluttered layout, perspective distortion in orthographic views, harsh shadows
```

## 7. Self-check before delivering

- The object is identical in every panel (same parts, same colors, same proportions).
- Orthographic views are truly front/side/back/top, not 3/4 angles, and share one scale.
- The scale label matches the brief, and the stud conversion is in the build spec.
- Exactly 4 callouts, 6 close-ups, 4 materials; hex codes in the prompt match the Materials panel.
- If an image was generated: read every label; image models often misspell text. If the text is broken,
  say so and suggest regenerating, or generating the panels without text and adding labels in an editor.

For two complete worked examples (the pouch and the dice that define this layout), read
`references/examples.md`.
