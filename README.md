# Process Scale-up Helper

Static, browser-based calculators to support mixing scale-up. No login or backend required.

## What it can be used for
- Early-stage scale-up estimates for agitated tanks.
- Comparing constant P/V vs constant tip-speed approaches.
- Quick checks of Reynolds number regime and torque.

This is a prototype. Validate inputs, units, and results before design or operations use.

## How to use
1. Open `index.html` in a browser.
2. Choose a scale-up rule:
   - Keep P/V constant: preserves power per unit volume from lab to plant.
   - Keep tip speed constant: preserves shear/tip-speed effects.
3. Enter your lab and plant inputs (units are shown on each field).
4. Review calculated outputs, including Reynolds regime.
5. For impeller guidance, open `np-guide.html`.

## Units used
- Density: g/mL
- Viscosity: Pa*s
- Impeller diameter: mm
- Volume: L
- Power: W
- P/V: W/m3
- Speed: rpm (inputs), 1/s (outputs)

## Files
- `index.html`: Mixing scale-up calculators.
- `np-guide.html`: Impeller type and Np guidance.
- `imgs/`: Impeller reference images.

