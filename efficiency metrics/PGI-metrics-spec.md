# PGI Metrics Page Spec

## Goal

Add a new browser-only page to this project that reads an uploaded Excel workbook,
extracts process data from the `PGI` sheet, lets the user select a `TP` block and
metrics, and calculates the selected values locally in the browser.

Target metrics for the first implementation:

- `PMI`
- `E-factor`
- `RME`

`AE` should be shown in the UI as `coming soon` or `experimental` until the data
model for solution reagents and reactive-input classification is finalized.

## Workbook Scope

Expected workbook structure:

- `PGI` sheet must exist.
- `Chemical data ref sheet` should exist when `RME` is calculated.

Supported file types:

- `.xlsx`
- `.xlsm`

Macros are not executed. Only worksheet cell values are read.

## PGI Sheet Model

The `PGI` sheet is treated as one structured table with columns `A:AR`.

Relevant columns:

- `A`: `Reaction step`
- `B`: `Intermediate product`
- `C`: `limiting reactant`
- `D`: `Chemical name`
- `E`: `CAS`
- `F`: `Comments`
- `H`: `molar mass (g/mol)`
- `O`: `Avg yield`
- `P`: `Product retention samples (g)`
- `S`: `mass (Main reactant of TP)`
- `T`: `Number of kilomoles (n)`
- `U`: `Output mass (kg)`; use as the per-row resolved batch mass
- `Y`: `Mass (available in next reaction step)`; use as net carry-forward mass

Important implementation note:

- Even though the header says `Output mass (kg)`, column `U` contains the resolved
  mass for ordinary input rows as well as product rows.
- For this page, treat column `U` as the row mass used in calculations.

## TP Block Detection

Distinct `TP` options are derived from non-empty values in column `A` that start with
`TP`.

A single TP block is all contiguous rows with the same `Reaction step`.

Example from the sample workbook:

- `TP.1` spans rows `20:30`
- `TP.4` spans rows `32:42`
- `TP.9` spans rows `83:88`

## Product Row Rule

Within a selected TP block:

- `product row` = the last row in the block where column `B` (`Intermediate product`)
  is non-empty

Reason:

- The first non-empty `B` row in a block may define the incoming main material.
- The last non-empty `B` row is the actual TP result row.

## Input Row Rule

Within a selected TP block:

- `input rows` = all rows before the `product row`
- include the top main-reactant row if it belongs to the TP block
- exclude blank separator rows

Also track the TP starting material from the workbook's main-reactant fields:

- use the first non-empty value from column `C` (`limiting reactant`) as the TP's
  carried-in starting-material code
- use column `S` (`mass (Main reactant of TP)`) as the corresponding carried-in mass
- if that starting material is not already present as an explicit input row in the TP
  block, add it as a synthetic `Starting material` row in the balance and row details

## Output Mass Rule

Product mass basis for all metrics:

- `gross product mass` = column `U` from the product row

Reason:

- column `Y` reflects carry-forward mass after retention samples or other deductions
- process metrics on this page should use product output before sample collection

## Metric Definitions

### PMI

Definition:

`PMI = total input mass / product mass`

Calculation:

- `total input mass` = sum of column `U` over all `input rows`
- include the carried-in starting-material mass from column `S` when it is not already
  represented by an explicit TP input row
- `product mass` = gross output from column `U` on the product row

### E-factor

Definition:

`E-factor = (total input mass - product mass) / product mass`

Equivalent form:

`E-factor = PMI - 1`

Calculation should reuse the same gross-output basis as `PMI`.

### RME

Definition for this page:

`RME = product mass / reactive input mass`

Where:

- `product mass` = gross output from column `U` on the product row
- `reactive input mass` = sum of column `U` over rows classified as reactive inputs
- include the tracked carried-in starting-material row in reactive input mass

Because the workbook does not encode full chemistry semantics cleanly, `RME` must use
explicit assumptions.

## RME Classification Rule

Default rule for `RME`:

Include a row in `reactive input mass` when all of the following are true:

- row is an `input row`
- column `U` is present and greater than 0
- chemical is not the product row
- chemical type is not `Solvent`
- comment does not match obvious non-reactive workup phrases

Use `Chemical data ref sheet` by CAS lookup to get `Type`.

Default exclusions:

- `Type = Solvent`
- comments containing:
  - `wash`
  - `washing`
  - `co-evap`
  - `co-evop`
  - `solvent exchange`
  - `for quench`
  - `buffer`
  - `retention sample`

Default inclusions:

- main starting material row
- rows with `Type = Other`
- rows with `Type = Acid`
- rows with `Type = Base`
- rows with `Type = Salt`

UI note:

- `RME` output should display the exact rows included in the denominator so the user
  can verify the assumption.

## AE Status

`AE` is not reliable enough for automatic calculation from `PGI` alone in v1.

Reason:

- some reactive inputs are solution reagents described only in free text, for example
  `50% water solution` or `35%`
- the sheet does not provide a consistently structured purity/concentration field
- automatic stoichiometric extraction would require heuristic parsing of comments

Recommended v1 behavior:

- keep `AE` disabled, or
- mark it `experimental`

Recommended v2 approach:

- allow manual reactant selection
- allow optional purity/concentration overrides
- then compute:
  - `AE = MW(product) / sum(stoich reactant MWs) * 100`

## Parsing Flow

1. User uploads workbook.
2. Read workbook client-side.
3. Validate that `PGI` exists.
4. Parse `PGI` rows into JSON records.
5. Build distinct TP list from column `A`.
6. When a TP is selected:
   - identify the TP row range
   - identify the product row
   - identify input rows
7. If `RME` is requested:
   - load `Chemical data ref sheet`
   - build CAS-to-Type lookup
   - classify rows
8. Calculate selected metrics.
9. Render summary plus row-level details.

## UI Flow

Page sections:

1. File upload
2. Workbook status
3. TP selector
4. Metric checkboxes
5. Assumptions panel
6. Calculate button
7. Results
8. Copy / save actions

Recommended controls:

- file input
- TP dropdown
- checkboxes:
  - `PMI`
  - `E-factor`
  - `RME`
  - `AE (experimental)`
- assumptions:
  - `RME reactive input rule`:
    - `Exclude solvents only`
    - `Exclude solvents and workup rows` default

## Result Output

For each calculated metric, show:

- metric name
- value
- product output basis used
- TP identifier

Also show a details table with:

- row number
- chemical name
- CAS
- molar mass, when the row is not a solution-style reagent entry
- comments
- row mass from `U`
- inclusion flag for each metric, where relevant

## Export Options

Provide:

- `Copy summary`
- `Copy details`
- `Download CSV`
- `Download JSON`

## Example Rule Check

Using the sample workbook and `TP.1`:

- input mass = sum of `U` for rows `20:29`
- gross product mass = `U30`

This yields approximately:

- `PMI (gross)` = `38.32`
- `E-factor (gross)` = `37.32`
- `RME (gross)` = `34.69%`

## Recommended Implementation Order

1. Build upload and TP parsing
2. Implement `PMI`
3. Implement `E-factor`
4. Implement `RME` with explicit row listing
5. Add copy/save features
6. Revisit `AE`
