# Corner RGB Floor Lamp (`light.sypialnia_floor_lamp_corner`) Integration Ideas

This document outlines 3 concepts for integrating the bedroom corner RGB floor lamp (`light.sypialnia_floor_lamp_corner`) with the existing **IKEA BILRESA Dual Button** remote (`I` / `O`).

---

## Idea 1: Multi-Layer Bedroom Scenes (ÅRSTID Bedside + RGB Floor Accent)
Instead of controlling only the bedside lamps, the scenes under Button **`I`** set the mood across the entire bedroom using the RGB floor lamp as ambient background lighting:

* **`I` (1× click — Afternoon):**
  * ÅRSTID bedside lamps (`light.bedroom_bedside_lamps`): `80%`, `3300K`
  * Corner floor lamp (`light.sypialnia_floor_lamp_corner`): `70%`, `3000K`
* **`I` (2× click — 4-Scene Rotation with RGB Accents):**
  1. **Evening:** ÅRSTID `50% / 2700K` + Floor lamp in **warm sunset / amber** (`50%`)
  2. **Reading:** ÅRSTID `80% / 3500K` + Floor lamp `60% / 3500K`
  3. **Relax (Chill / Ambient):** ÅRSTID `15% / 2200K` + Floor lamp in **ambient RGB color** (e.g., deep indigo/violet or warm coral, `40%`)
  4. **Corner Only (Movie / Desk mode):** ÅRSTID lamps **turn off**, leaving only the corner floor lamp on (`50%`)
* **`O` (1× click):** Smoothly turns off **all bedroom lights** (both ÅRSTID bedside lamps and the corner floor lamp, `transition: 0.8s`).
* **`O` (2× click — Night Mode):** Turns off the floor lamp and sets the bedside lamps to `3% / 2200K`.

---

## Idea 2: Expand `I + O` Combo with a 3rd Mode $\rightarrow$ Continuous RGB Color Wheel Sweep
Currently, clicking **`I + O` together** toggles `input_select.bedroom_hold_mode` between:
1. **Brightness** (ÅRSTID bedside lamps)
2. **Color Temperature** (ÅRSTID bedside lamps)

Adding a 3rd option to the `I + O` cycle:
3. **Floor Lamp RGB**:
   * Switching into this mode triggers a brief confirmation pulse on the **floor lamp** so you know you are controlling the corner zone.
   * **Hold `I`:** Smoothly sweeps through the **RGB color wheel (Hue `0° -> 360°`)** on the floor lamp — releasing the button freezes it on the exact color you want.
   * **Hold `O`:** Smoothly dims/brightens the floor lamp.

---

## Idea 3: Split Button Roles (Top = Bedside, Bottom 2× Click = Floor Lamp)
Keep the floor lamp completely independent from the bedside scenes:
* **`I` (1× click, 2× click):** Controls only the ÅRSTID bedside lamps (as currently configured).
* **`O` (2× click):** Cycles through predefined RGB colors on the floor lamp (*Warm White $\rightarrow$ Amber $\rightarrow$ Violet $\rightarrow$ Floor Lamp Off*).
* **`O` (1× click):** Turns off all active bedroom lights.
