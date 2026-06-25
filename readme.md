# zmk-config-iso74

ZMK firmware for **iso74** — a 74-key handwired unibody keyboard on a **Supermini
nRF52840** controller (nice!nano v2 pin-compatible). Matrix is **6 rows × 15 columns**,
one diode per switch on the column-side leg, stripe toward the column bus (`diode-direction = "row2col"`).

The matrix is fully validated and the **real keymap** is in place (`iso74.keymap`):
3 layers — base QWERTY/ISO, `fn1` (F1-F12, bluetooth/output/bootloader, media on the
arrows), and `fn2` (blank, yours to fill). The `default_transform` in `iso74.overlay`
remaps the scrambled column wiring to the real layout (derived from the discovery scan).

## Solder map — wire the matrix to these exact pads

Rows (6) — all grouped on the left edge for clean bus routing:

| matrix | pad | nRF   |
|--------|-----|-------|
| row0   | 006 | P0.06 |
| row1   | 008 | P0.08 |
| row2   | 017 | P0.17 |
| row3   | 020 | P0.20 |
| row4   | 022 | P0.22 |
| row5   | 024 | P0.24 |

Columns (15):

| matrix | pad | nRF   | note |
|--------|-----|-------|------|
| col0   | 100 | P1.00 |      |
| col1   | 011 | P0.11 |      |
| col2   | 104 | P1.04 |      |
| col3   | 106 | P1.06 |      |
| col4   | 031 | P0.31 |      |
| col5   | 029 | P0.29 |      |
| col6   | 002 | P0.02 |      |
| col7   | 115 | P1.15 |      |
| col8   | 113 | P1.13 |      |
| col9   | 111 | P1.11 |      |
| col10  | 010 | P0.10 | NFC  |
| col11  | 009 | P0.09 | NFC  |
| col12  | 101 | P1.01 |      |
| col13  | 102 | P1.02 |      |
| col14  | 107 | P1.07 |      |

`col10`/`col11` use the NFC pins, freed via `&uicr { nfct-pins-as-gpios; };` in
`iso74.overlay`. To change any assignment, edit the `row-gpios`/`col-gpios`
arrays in `config/boards/shields/iso74/iso74.overlay`.

> ⚠️ **Two different column numberings — don't mix them up.**
> The `col0`–`col14` in the table above are **firmware/pin indices**: `col0` just means
> "the column pin at pad P1.00". 0-indexed, ordered by pad, NO inherent physical meaning.
> The `phys-col 1`–`15` in the population table below are **physical key columns counted
> right→left**. Which physical column you solder to which pad is your free choice — just
> record it; the matrix transform maps firmware `(row,col)` → the real key. (For the
> single-switch test the physical 5th-from-right column happened to be wired to pad P1.00
> = firmware `col0`; that pairing was arbitrary.)

## Real matrix population (reference for the final transform)

Which switches actually exist, as wired. **Physical** columns numbered **right→left**
(phys-col 1 = rightmost key column), rows **top→bottom** (row1 = top). 74 switches.

| phys-col | rows present  | count |
|-----|---------------|-------|
| 1   | 2,3,4,5       | 4 |
| 2   | 2,3,4,5,6     | 5 |
| 3   | 2,4,5         | 3 |
| 4   | 2,3,4,5,6     | 5 |
| 5   | 1,2,3,4,5,6   | 6 |
| 6   | 1,2,3,4,5     | 5 |
| 7   | 1,2,3,4,5     | 5 |
| 8   | 1,2,3,4,5     | 5 |
| 9   | 1,2,3,4,5,6   | 6 |
| 10  | 2,3,4,5       | 4 |
| 11  | 1,2,3,4,5     | 5 |
| 12  | 1,2,3,4,5     | 5 |
| 13  | 1,2,3,4,5,6   | 6 |
| 14  | 1,2,3,4,5,6   | 6 |
| 15  | 2,3,4,5       | 4 |

Per-row totals: row1=9, row2=15, row3=14, row4=15, row5=15, row6=6.

The **ISO enter** switch sits at **col3 / row4**; **col3 / row3 is empty** — the top half
of the tall enter keycap (the gap in the QWERTY row).

The current firmware uses the FULL 6×15 grid for bring-up, so all 74 real switches register
and the 16 empty intersections are simply never pressed — **no change needed to test**.
Use this table afterward to trim `default_transform` to the real 74-key layout.

## Build & flash

1. Push this repo to GitHub. The Actions workflow builds automatically.
2. Actions tab → latest run → download the **firmware** artifact → `iso74 nice_nano.uf2`.
3. Double-tap reset on the controller → it mounts as `NICENANO` → drag the `.uf2` onto it.

## Test the matrix

Plug in over USB, open a text editor, and short each switch (or each row↔column crossing
with tweezers). Every crossing should type its unique character per the grid in
`iso74.keymap`. Walk the whole grid to confirm all 6 rows and 15 columns scan, with no
dead lines and no doubled keys.

- **Nothing registers at all** → diodes are oriented the other way: change
  `diode-direction` to `"col2row"` in `iso74.overlay` (and move the pull-down back to
  the row-gpios), rebuild (no resolder).
- **A whole row/column is dead** → check that pin's solder joint / magnet-wire enamel
  stripping.
