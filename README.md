# Corne-ish Zen v2 — Miryoku + ZMK Studio

A native ZMK config for the **Corne-ish Zen v2 (GB R3)** based on Matburt's repo that reproduces the
Miryoku layout (changed to a Colemak-DH Mod base) **and** is editable live in [ZMK Studio](https://zmk.studio/).
I've also added back the zmk-display-patch tree for the Miryoku logo.

The keymap is transcribed directly from the `manna-harbour/miryoku_zmk` generated
layers (default / right-hand-nav), with a **QWERTY** base instead of the stock
Colemak Mod-DH — but written as a plain keymap so Studio can read and edit it. Unlike the
`miryoku_zmk` repo (a compile-time C-macro system Studio can't parse), you rebuild
once and then move keys in the browser.

## Files

```
build.yaml                     # builds both halves; Studio snippet on the left
config/corneish_zen.conf       # Kconfig: Studio, pointing, display
config/corneish_zen.keymap     # the Miryoku layout as a native keymap
```

## How to use it

1. Create a GitHub repo (e.g. `zmk-config-zen`) and drop these files in — `build.yaml`
   at the root, the other two in `config/`.
2. Commit and push. GitHub Actions builds automatically (enable Actions if prompted).
3. Download the artifact zip from the run. You'll get `corneish_zen_left`,
   `corneish_zen_right`, `corneish_zen_left_studio` (Studio-enabled left), and
   a settings-reset UF2.
4. Flash over USB — double-tap reset, drag the matching UF2 onto the drive. Flash the
   **right** UF2 to the right half. For the left, use **`corneish_zen_left_studio`**
   if you want live editing, or the plain **`corneish_zen_left`** if you don't.

> Best base to start from: the [ZMK new-user template](https://github.com/zmkfirmware/unified-zmk-config-template)
> (pick Corne-ish Zen v2), or fork [LOWPROKB/zmk-config-zen-2](https://github.com/LOWPROKB/zmk-config-zen-2),
> then replace its `build.yaml` and `config/` with these files.

## Studio support: confirmed working

The Zen v2 board **ships physical layouts** — verified against LowproKB's official
zen-2 config and ZMK's `corne.dtsi`. So Studio works out of the box; no overlay needed.
This config is set up for the **5-column (36-key)** Zen:

- **Selects `zmk,physical-layout = &foostan_corne_5col_layout`** in the keymap, and the
  keymap has 36 bindings per layer to match. (For a 6-col/42-key board, switch to
  `&foostan_corne_6col_layout` and add the outer-column keys back.)
- **Does NOT pin `zmk,matrix-transform`** (the old, Studio-incompatible way the stock
  Miryoku Zen keymap used). The physical layout carries the transform now.
- **Enables Studio per-build** via `cmake-args` on a dedicated `*_studio` artifact in
  `build.yaml`, exactly like LowproKB — keeping the normal firmware lean.

Filenames (`corneish_zen.keymap` / `.conf`) match LowproKB's config, so the build
will find them.

## Using ZMK Studio

1. Flash `corneish_zen_left_studio` to the left half and connect it over **USB**.
2. Open <https://zmk.studio/> in Chrome or Edge and connect.
3. **Unlock editing:** press both **outer thumb keys together** (the `&studio_unlock`
   combo). Now you can rearrange keys and layers.
4. Edits persist on the keyboard; a settings reset returns it to this compiled base.

You only rebuild firmware for changes to combos, home-row-mod timing, the Bluetooth
morph, or `.conf`. Everyday key swaps happen in Studio.

> ### ⚠️ Pick one source of truth
>
> There are **two independent places** the layout can live, and they do **not** sync:
>
> - **This repo** (`config/corneish_zen.keymap`) — what CI compiles into the firmware.
> - **The keyboard's own storage** — where ZMK Studio writes your live edits.
>
> They can drift apart. If you edit in Studio and later flash a fresh build from
> this repo, **the build wins and your Studio changes are gone**. If you edit the
> keymap here but keep using Studio-saved changes on the board, the repo no longer
> reflects what you actually type.
>
> Decide which one is authoritative:
>
> - **Repo as source of truth** (recommended for anything you want to keep): make
>   changes in `corneish_zen.keymap`, push, and reflash. Treat Studio as scratch —
>   a settings reset returns the board to exactly what's compiled here.
> - **Studio as source of truth**: do your remapping in Studio and **avoid
>   reflashing**, since any new build silently overwrites the board. If you go this
>   route, periodically transcribe changes back into the keymap so the repo isn't
>   stale — the two never reconcile automatically.

## What's in the layout (faithful Miryoku)

All eight layers are ports of stock Miryoku, with a QWERTY base layer:

- **Base** — QWERTY with home-row mods (GUI/Alt/Ctrl/Shift), AltGr mod-taps on
  X and `.`, and a Button-layer hold on Z and `'`.
- **Nav / Mouse / Media** — held from the left thumbs; functions on the right hand,
  mods on the left. Nav has arrows + clipboard + INS/HOME/PGUP/PGDN/END. Mouse uses
  ZMK pointing (`&mmv` move, `&msc` scroll, `&mkp` buttons). Media has volume/track
  transport plus Bluetooth.
- **Num / Sym / Fun** — held from the right thumbs; functions on the left hand, mods
  on the right.
- **Button** — clipboard + mouse buttons, reached by holding Z or `'`.

Deviations from stock, all deliberate:

- **Home-row mods are positional** (`hml`/`hmr`) instead of plain mod-taps — fewer
  misfires. Tune `tapping-term-ms` / `flavor` at the top of the keymap.
- **Extra and Tap alternate base layers are omitted** — the "switch base" keys on the
  number/sym/fun layers that pointed to them are left as `&none`. "Return to Base"
  and per-layer lock keys still work.
- **RGB / external-power keys are `&none`** — the Zen has nice!view displays, no RGB
  underglow, so those Miryoku keys have nothing to drive.

### Clipboard
Set to Miryoku's default (CUA / Windows-Linux: `K_AGAIN`, `LS(INS)`, `LC(INS)`,
`LS(DEL)`, `K_UNDO`). On **macOS** these won't behave — swap the `U_RDO…U_UND`
defines near the top of the keymap to the `LG(...)` set shown in the comment there.

## Bluetooth: clear a profile / pair a new computer

On the **Media** layer (hold the left outer thumb), the bottom-right row is your
Bluetooth row: `OUT_TOG` then profiles 0–3.

- **Tap** a profile key → select/switch to that profile.
- **Shift + tap** a profile key → **clear** that profile's pairing (the keys are
  shift-morphs, exactly like stock Miryoku).

To move to a new computer: shift+tap the profile to wipe it, tap it to select
(it starts advertising), then pair from the new computer. Do it from the left half.
If pairing wedges, flash the **settings-reset UF2** to both halves, forget the
keyboard everywhere, and reflash normal firmware.
