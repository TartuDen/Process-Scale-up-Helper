# Process Scale-up Helper

Static, browser-based calculators for mixing and heating scale-up checks. No login or backend required.

## What it can be used for
- Early-stage scale-up estimates for agitated tanks.
- Comparing constant P/V vs constant tip-speed approaches.
- Quick checks of Reynolds number regime and torque.
- UA and exotherm checks for heating scale-up.

This is a prototype. Validate inputs, units, and results before design or operations use.

## How to use
1. Open `index.html` in a browser.
2. Pick a calculator:
   - Stirring scale-up (`mixing.html`)
   - Heating scale-up / UA (`heating.html`)
3. Enter your inputs (units are shown on each field).
4. Use the Tutorial button for guided, field-by-field help.
5. Review calculated outputs and regime checks.
6. For impeller guidance, open `np-guide.html`.

## Units used
- Density: g/mL
- Viscosity: Pa*s
- Impeller diameter: mm
- Volume: L
- Power: W
- P/V: W/m3
- Speed: rpm (inputs), 1/s (outputs)

## Files
- `index.html`: Landing page with tool links.
- `mixing.html`: Mixing scale-up calculators.
- `heating.html`: UA and exotherm check calculator.
- `np-guide.html`: Impeller type and Np guidance.
- `imgs/`: Impeller reference images.
