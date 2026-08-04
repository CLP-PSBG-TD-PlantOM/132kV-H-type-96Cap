# 132kV H Type 96 Capacitor Bank Unbalanced Current Optimization

Static web tool for calculating and reducing unbalanced current in a 132kV H type capacitor bank.

## Final Website

Open the published GitHub Pages site here:

https://clp-psbg-td-plantom.github.io/132kV-H-type-96Cap/

## Purpose

This tool helps users:

- enter measured capacitance values for a 96-capacitor 132kV H type capacitor bank
- calculate primary unbalanced current and secondary relay current
- compare L1, L2, and L3 phases in one web page
- load measured data from Excel or CSV
- generate swap recommendations to reduce unbalanced current
- export the recommended swap result and final layout as CSV

## Capacitor Bank Configuration

The model uses 96 capacitors split into four H-bridge sections:

| Section | Physical position | Capacitor numbers |
| --- | --- | --- |
| C1 | Front top | 1-24 |
| C2 | Front bottom | 25-48 |
| C3 | Rear top | 49-72 |
| C4 | Rear bottom | 73-96 |

Each section contains:

- 24 capacitors
- 6 groups in series
- 4 capacitors in parallel per group

The visual guide shows two separated 6-tier towers:

- Front tower: C1 and C2, capacitors 1-48
- Rear tower: C3 and C4, capacitors 49-96

On mobile screens, the visual guide is horizontally scrollable so the full Front and Rear layout can still be viewed.

## How To Use

1. Open the website.
2. Select the phase tab: L1, L2, or L3.
3. Enter or load the 96 measured capacitance values for the selected phase.
4. Check the calculated primary and secondary unbalanced current.
5. Choose the swap mode.
6. Click **Optimize swaps**.
7. Review the recommended swap pairs.
8. Click **Apply** if the recommendation should be applied to the current phase layout.
9. Click **Export CSV** to save the result.

Each phase stores its own values and optimization result while the page remains open.

## Controls And Buttons

| Control | Function |
| --- | --- |
| System voltage kV | Sets the line-to-line system voltage. Default is 132 kV. |
| Frequency Hz | Sets system frequency. Default is 50 Hz. |
| Nominal capacitance uF | Sets the default capacitance value used when resetting or creating template data. Default is 21.89 uF. |
| CT ratio X:1 | Sets the CT ratio used to calculate secondary relay current from primary unbalanced current. Default is 1:1. |
| Swap pairs | Selects a fixed number of swap pairs, or Auto mode. |
| Load Current Phase | Loads Excel, CSV, or TXT capacitance data into the currently selected phase only. |
| Download Template | Downloads an Excel template if Excel support is available, otherwise downloads a CSV template. |
| Example | Loads example capacitance values into the selected phase for testing. |
| Reset Phase | Resets the selected phase to the nominal capacitance value and clears optimization output. |
| L1 / L2 / L3 | Switches between the three phase datasets. |
| Optimize swaps | Searches for capacitor swap recommendations for the selected phase. |
| Apply | Applies the current swap recommendation to the selected phase layout. |
| Export CSV | Exports the optimization summary, swap list, and final layout. |

## Excel Or CSV Import Format

The recommended import format is the template produced by **Download Template**:

| ID | Section | Group | Slot | Capacitance_uF |
| --- | --- | --- | --- | --- |
| 1 | C1 | G1 | 1 | 21.89 |
| 2 | C1 | G1 | 2 | 21.89 |
| ... | ... | ... | ... | ... |
| 96 | C4 | G6 | 4 | 21.89 |

Import logic:

- If rows contain capacitor ID and capacitance value, IDs 1-96 are used to place values correctly.
- If no usable ID mapping is found, the first 96 numeric values are loaded in order.
- The file is loaded into the currently selected phase only.
- Supported file types: `.xlsx`, `.xls`, `.csv`, `.txt`.

## Calculation Logic

The calculator uses an approximate short-circuit H-bridge model for the relay / CT branch.

- Each group is four capacitors in parallel.
- Each section is six parallel groups in series.
- Voltage is calculated as line-to-neutral voltage: `system kV / sqrt(3)`.
- The displayed unbalanced current is the primary-side current unless marked as secondary.
- Secondary relay current is primary unbalanced current divided by the CT ratio.

```text
G = cap1 + cap2 + cap3 + cap4
Csection = 1 / (1/G1 + 1/G2 + 1/G3 + 1/G4 + 1/G5 + 1/G6)
V = system kV / sqrt(3) * 1000
Ctop = C1 + C3
Cbottom = C2 + C4
Ctotal = Ctop * Cbottom / (Ctop + Cbottom)
Itotal = V * 2 * pi * frequency * Ctotal * 1e-6
Iunbalance_primary = abs(I1 - I2)
Iunbalance_secondary = Iunbalance_primary / CT ratio
```

Capacitance values are entered in uF. The JavaScript calculation converts uF to F when calculating current.

## Swap Optimization Logic

The optimizer searches capacitor swaps to reduce primary unbalanced current.

### Fixed pair mode

If a fixed number of swap pairs is selected, the optimizer tries to find the best sequential improvement up to that number of pairs.

### Auto mode

Auto mode searches up to 6 swap pairs and recommends the fewest useful number of pairs.

Auto stops when:

- the displayed result reaches `0.000 mA`, or
- the next improvement is less than `0.001 mA`, or
- 6 swap pairs have been checked

This avoids recommending extra swaps that do not provide meaningful improvement.

### Swap display

Each recommended pair shows:

- capacitor ID
- original position
- target position
- measured capacitance value
- calculated unbalanced current after that pair

The matching pair colors in the table identify which two capacitors belong to the same swap.

## Output Values

| Output | Meaning |
| --- | --- |
| Primary unbalanced current | Calculated unbalanced current on the primary side. |
| Secondary relay current | Primary unbalanced current divided by CT ratio. |
| After swaps primary | Primary unbalanced current after applying the recommended swap layout. |
| After swaps secondary | Secondary relay current after applying the recommended swap layout. |
| Calculation Details | Shows effective voltage and C1/C2/C3/C4 equivalent capacitance. |

## Export CSV Contents

The exported CSV includes:

- phase name
- date and time
- voltage, frequency, and CT ratio
- swap mode
- recommended swap pair count
- before and after primary unbalanced current
- before and after secondary relay current
- improvement percentage
- full swap list
- final capacitor layout

## Notes

- This is a browser-only static web application.
- No backend server is required.
- The relay / CT branch is modeled using an approximate short-circuit H-bridge calculation.
- Users should verify final swap decisions against site procedures, protection settings, and engineering requirements before field implementation.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | Page structure and controls. |
| `styles.css` | Responsive layout and visual styling. |
| `app.js` | Calculation, import/export, optimization, phase handling, and SVG visual guide. |
