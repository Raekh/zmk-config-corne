# Notes

## Build: ZMK pinned to v0.3

ZMK `main` moved to Zephyr 4.1, which broke the build with `Invalid BOARD` for
`nice_nano_v2`. ZMK is now pinned to `v0.3` in `config/west.yml` and
`.github/workflows/build.yml`.

Upgrading to Zephyr 4.1 later would need two things:
- Rename the board in `build.yaml` from `nice_nano_v2` to `nice_nano//zmk`
  (all entries, including `settings_reset`).
  See https://zmk.dev/blog/2025/12/09/zephyr-4-1
- Port `nice-view-mod` to LVGL 9. It uses LVGL 8 APIs that were removed
  (`lv_canvas_draw_rect`, `LV_IMG_CF_TRUE_COLOR`, `lv_draw_img_dsc_init`).

## Accented characters (not implemented yet)

### Problem
The accent layer uses unicode macros (`Ctrl+Shift+U` + hex code). That only
works in GTK/IBus apps on Linux (not Qt, terminals, some Electron apps) and not
at all on Windows.

### Options considered
- **US International**: rejected. It makes `'` `"` `` ` `` `^` `~` dead keys.
- **"English (intl., with AltGr dead keys)"**: keeps quotes normal, but it only
  exists on Linux. Windows would need a different accent layer.
- **Windows Alt codes** (Alt+0233...): depend on NumLock and are unreliable in
  some apps. Last resort only.
- **EurKEY**: chosen candidate. It's based on US, keeps `'` `"` `` ` `` `^` `~`
  as normal keys, and puts accents behind AltGr (= Right Alt, `RA(...)`).
  It has the same AltGr positions on Linux (xkb `eu`) and Windows (v1.3).

### EurKEY mapping (checked against /usr/share/X11/xkb/symbols/eu and the official Windows v1.3 PDF)

| Char | Keycodes to send |
|---|---|
| é | `RA(G)` |
| è | `RA(F)` |
| à | `RA(Z)` |
| ù | `RA(H)` |
| ç | `RA(C)` |
| ë | `RA(E)` |
| ï | `RA(I)` |
| ê â ô î û | `RA(N6)` (dead circumflex), then the letter |
| œ | `RA(P)` |
| « » | `RA(LBKT)` / `RA(RBKT)` |

Capitals: Shift + AltGr + key for the one-keystroke ones. For the circumflex,
Shift must NOT be held during `RA(N6)` (Shift+AltGr+6 = dead caron), only on
the letter.

### Plan
- Rebuild the "Unicode (L)" layer (layer 5) with the same characters in the
  same positions, using the mapping above. Rename it "Accents".
- Remove the unicode macros and `config/keys_us_international.h`.
- Base and other layers unchanged.

### OS setup (once per computer)
- Linux: add keyboard layout "EurKEY (US)".
- Windows: install EurKEY 1.3 from https://eurkey.steffen.bruentjen.eu/download.html
  (unofficial installer with ARM64 support:
  https://github.com/rpnfan/EurKey-fork-for-Windows), then select it in
  language settings.
- On a computer not set to EurKEY, the accent keys send the wrong thing
  (e.g. Alt+G).

### Open questions
- Add Shift capitals (É À Ç...)?
- Add œ « »?
