# DAX Colour Control Demo — Power BI

A demonstration of dynamic, data-driven map colouring in Power BI using pure DAX. No custom visuals, no external tools. Each SA3 region on the map is painted a colour calculated entirely in DAX based on a similarity score and an assigned colour family.

This file was built to accompany a LinkedIn post on conditional formatting techniques for geographic maps in Power BI.

---

## What It Does

The report loads Australian SA3 boundary data alongside a randomly generated dataset that assigns each SA3 region:

- A **colour family** (Green, Blue, Yellow, or Purple)
- A **degree of similarity score** from 0 (not similar) to 5 (similar)

Three DAX measures then work together to drive the visual experience:

1. `_ColorMeasureAustralia` colours each region on the map dynamically based on its score and family
2. `ButtonsDefaultColour` colours the filter buttons to match their assigned family
3. `ButtonsHoverColour` provides a lighter shade of the same family for hover/selected state

---

## Data Sources

### Map Boundaries

**Australian Bureau of Statistics (ABS) — ASGS Edition 3 (2021)**

Statistical Area Level 3 (SA3) digital boundary files, published 20 July 2021.

- Format: GeoPackage / ESRI Shapefile (GDA2020 / GDA94)
- Coverage: Geographic Australia
- Licence: Creative Commons Attribution 4.0 International (CC BY 4.0), Copyright Commonwealth of Australia administered by the ABS
- Download: [ABS Digital Boundary Files](https://www.abs.gov.au/statistics/standards/australian-statistical-geography-standard-asgs-edition-3/jul2021-jun2026/access-and-downloads/digital-boundary-files)

SA3 regions are designed for regional data output and typically cover populations between 30,000 and 130,000 people. They sit in the middle of the ASGS hierarchy: above SA2s (community level) and below SA4s (labour market level).

### Similarity Score Data

A randomly generated Excel file created for demonstration purposes only. It contains:

- SA3 codes and names matching the ABS boundary file
- An assigned colour family per SA3 (Green, Blue, Yellow, or Purple)
- A degree of similarity score (integer, 0 to 5) per SA3

No real-world data is used. The scores have no statistical meaning.

---

## The Three Measures

### 1. `_ColorMeasureAustralia`

The core measure. Returns a hex colour string (e.g. `#3D9A4F`) for each SA3 region, which Power BI uses in conditional formatting to paint the map.

**How it works:**

Each colour family has a light starting colour (score = 0) and a deep target colour (score = 5). The measure interpolates linearly between them using the formula:

```
Channel = BaseValue - ((score / 5) * Reduction)
```

This is done independently for the Red, Green, and Blue channels. The three resulting decimal values (0–255 each) are then converted to 2-digit hex manually, because DAX has no native decimal-to-hex function. The conversion uses a character lookup string `"0123456789ABCDEF"` and `MID()` to extract each digit.

**Edge cases:**

| Condition | Output | Reason |
|---|---|---|
| `val` is blank | `#E6E6E6` (grey) | Row has no score, e.g. map totals context |
| `val = 0` | `#F5F9CE` (cream) | Explicitly "not similar", visually distinct from colour families |
| `val` 1–5 | Dynamic hex | Interpolated colour within the assigned family |

**Colour family reference (light → deep):**

| Family | Score 0 (approx.) | Score 5 (approx.) |
|---|---|---|
| Green | `#CDFF11` (lime) | `#009200` (deep green) |
| Blue | `#E0F6FE` (ice blue) | `#0D6ABF` (deep blue) |
| Yellow | `#FFF8C2` (cream yellow) | `#FFF800` (strong yellow) |
| Purple | `#E8E0FA` (lavender) | `#57573B` (deep purple) |

---

### 2. `ButtonsDefaultColour`

Returns a solid hex colour for each colour family. Used to set the background colour of filter buttons so they visually match the map regions they control.

```dax
SWITCH(
    TRUE(),
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Green",  "#66c875",
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Purple", "#9F8DC6",
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Blue",   "#76B0DE",
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Yellow", "#FCF961"
)
```

Each value is a fixed mid-tone chosen to represent the family clearly without being too dark for legible button text.

---

### 3. `ButtonsHoverColour`

Returns a lighter version of each family colour. Applied as the conditional formatting colour for the selected or hovered state of the filter buttons, giving visual feedback when a filter is active.

```dax
SWITCH(
    TRUE(),
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Green",  "#CDFFDF",
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Purple", "#E8E0FA",
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Blue",   "#E0F6FE",
    SELECTEDVALUE('SA3_Colour_Map'[Maps to]) = "Yellow", "#F9F8C2"
)
```

The hover colours are pastel equivalents of the default colours, keeping the same hue family but at low saturation so the selected state is obvious without being jarring.

---

## How to Use

1. Open `DAX Color Control Demo.pbix` in Power BI Desktop
2. The map visual uses `_ColorMeasureAustralia` under **Conditional Formatting > Fill colour**
3. The filter buttons use `ButtonsDefaultColour` and `ButtonsHoverColour` under their respective conditional formatting settings
4. Swap in your own data by replacing the Excel source, keeping the same column structure (SA3 code, colour family, similarity score)

---

## Licence

Map boundary data: CC BY 4.0, Copyright Commonwealth of Australia (ABS).
Demo data and DAX: free to use and adapt.
