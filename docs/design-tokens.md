# Tactile Web — Tokens

These are settled, not drafts. Arrived at by testing live in a comparison tool rather than picking from hex codes on a page.

## 1. Palette

Two modes, pick based on project preference or system setting, no forced default.

**Light — "Old browser gray" (deliberately unfashionable, borrows actual old browser chrome)**

| Role | Hex |
|---|---|
| Background | `#C4C4C0` |
| Surface / panel | `#D9D9D5` |
| Surface, pressed | `#ABABA6` |
| Border / structural line | `#000000` |
| Text primary | `#000000` |
| Text secondary | `#4A4A46` |
| Bevel light edge | `#FFFFFF` |
| Bevel dark edge | `#7A7A75` |

**Dark — "Slate ink" (charcoal, not the green-terminal cliché)**

| Role | Hex |
|---|---|
| Background | `#1C1C1A` |
| Surface / panel | `#262624` |
| Surface, pressed | `#333330` |
| Border / structural line | `#EDEAE2` |
| Text primary | `#EDEAE2` |
| Text secondary | `#A6A297` |
| Bevel light edge | `#3A3A36` |
| Bevel dark edge | `#0E0E0D` |

**Functional accents** (fixed per mode, same role every project):

| Role | Light (Old browser gray) | Dark (Slate ink) |
|---|---|---|
| Active / success | `#2E7D32` | `#4FBF6B` |
| Waiting / pending | `#B8791F` | `#E0A83A` |
| Warning / danger | `#A5342A` | `#E05B4C` |
| Link | `#1E5FA0` | `#5B9BE0` |
| Secondary system info | `#28848C` | `#3FC4C4` |

## 2. Raised / recessed mechanism — border bevel, deep recess

Not a drop shadow. The border itself carries the light/dark edges.

- **Idle (raised)**: top and left border edges use the bevel light color, bottom and right use the bevel dark color. Border width `2px`. No shadow.
- **Hover**: same as idle. No shadow, no shift.
- **Pressed (recessed)**: edge colors invert, top/left become bevel dark, bottom/right become bevel light. Border width increases to `3px`. Add `inset 3px 3px 5px rgba(0,0,0,0.4)`. The whole control shifts `translate(1px, 1px)`.
- **Recessed panel** (a panel that's permanently in the sunken state, never raised): use the pressed treatment above, permanently.

## 3. Border and radius

- Border width: `2px` idle, `3px` on press (see above). `1px` for plain dividers that aren't interactive controls.
- Radius: `0px`, everywhere, no exceptions. This was tested against 2px, 4px, and 8px and 0px won outright.

## 4. Typography

- UI font: **Albert Sans** (Google Fonts, needs a font import, it's not a system font)
- Monospace: **Courier New** (system font, no import needed)
- Base size `1rem`, restrained scale (1.125–1.2 ratio), no dramatic type scale.

## 5. Spacing

- Base unit: `4px`
- Scale: `4, 8, 12, 16, 24, 32, 48, 64`

Not put through the same live-testing pass as color/font/shadow, this is a reasonable default rather than something tested. Fine to adjust per project if it doesn't fit, doesn't need another comparison round to change.

## 6. Icons

Also not live-tested. Default approach: a small custom pixel icon set (16x16 grid, scaled 2x), specifically the chunky, low-resolution Windows 98-era look, not a generic modern pixel-art style, for a fixed core set (nav, status, a handful of primary actions), plain unbranded outline icons for everything else. Adjust per project as needed.

## 7. Dark mode and switching

Resolved: yes, both modes exist and are specified above (Old browser gray / Slate ink). Not an afterthought, a real second variant with its own bevel edge colors and accent values, not just inverted background and text on the light palette.

Default behavior: respect the OS-level `prefers-color-scheme` setting. Also include a persistent three-way control (Light / Dark / System) so the default can be overridden per user, not just per machine. Style this control using the bevel mechanism itself, a small segmented set of three bevel-buttons, whichever mode is active shows in the pressed/recessed state, the other two stay raised.

## 8. Focus state

Keyboard focus gets an outline in the link accent color, `2px`, offset `2px` outward from the control (`outline: 2px solid var(--accent-link); outline-offset: 2px;` or equivalent). This is a plain outline, not a box-shadow, it doesn't touch the no-drop-shadow rule. Non-negotiable for accessibility, don't skip this on any interactive control.

## 9. Disabled state

- Opacity reduced to ~50%
- Bevel flattened entirely: single plain border color (the normal border color, not the light/dark bevel split), no raised or recessed read at all
- `cursor: not-allowed`
- No hover or press response of any kind

## 10. Motion timing

Deliberately near-instant, not smoothed. Eased, gradually-animated button presses read as generic and AI-generated at this point, the whole point of a mechanical metaphor is that it snaps.

- Button/bevel state changes (idle, hover, pressed): effectively instant, `0ms`, no transition property at all
- Toggle knob slide: `~40ms`, just enough to not look like a rendering glitch, still reads as a snap rather than a glide
- No easing curves beyond linear where any duration exists at all
