> ⚠️ Deprecation notice  
> This repository is a **legacy** version of the SN-Rating-Model and is kept for
> historical and reference purposes only.  
> It is **deprecated** and not aligned with the latest model design, configuration
> or documentation.  
> For any new work, please use the repository:  
> (https://github.com/snlabs-tech/SN-Rating-Model)

# SN — Corporate Rating Model (V1 Prototype)

This repository contains a prototype corporate credit rating model implemented in Python, designed to combine quantitative financial ratios with qualitative risk assessments into a single, transparent issuer rating and outlook.

## Purpose and scope

- Demonstrate an end-to-end, rule-based corporate rating engine from raw inputs to final rating and outlook.
- Provide a clear, inspectable alternative to “black-box” ML models for corporate credit risk.
- Serve as a V1 baseline for experimentation, benchmarking, and documentation; a more refined V2 model supersedes this version.

## Core design

The model operates on a 0–100 internal score scale mapped to a 19-notch rating grid from AAA to CCC-, with an optional sovereign cap.  
It explicitly separates quantitative (financial) and qualitative (expert-judgment) components and combines them via configurable weights.

## Main components

### `SN_RatingComponents`

Structured dataclass that bundles all key outputs: quantitative score, qualitative score, combined score, uncapped rating, final rating (after sovereign cap), outlook, and a flag indicating if the sovereign cap was binding.

### `SN_CorporateRatingModel`

Rating engine class that orchestrates the full process:

- `compute_quantitative_score()`: maps leverage, coverage, profitability, and liquidity ratios to 1–5 levels and then to 0–100 scores using predefined threshold grids.
- `compute_qualitative_score()`: converts 1–5 expert scores on business risk, management/governance, and country/structural factors into the same 0–100 scale.
- `combine_scores()`: applies normalized weights to quantitative and qualitative blocks.
- `derive_outlook()`: assigns Positive / Stable / Negative based on where the combined score sits within the band of the implied rating.
- `rate()`: single entry point that runs the full pipeline and returns `SN_RatingComponents`.

### Top-level helper functions

- `level_score()`: generic mapper from raw ratio to level (1–5) and then to internal score via `LEVEL_TO_SCORE`.
- `score_to_rating()`: maps a combined score to a letter rating using the `RATING_GRID`.
- `rating_band()`: returns the score interval associated with a given rating for outlook logic.
- `apply_sovereign_cap()`: enforces a sovereign ceiling by capping the issuer rating at the sovereign level if needed.

## Inputs and outputs

- **Quantitative input**: dictionary of financial ratios (e.g. debt/EBITDA, EBITDA/interest, CFO/debt, margins, ROE/ROA, liquidity, maturity wall), expressed as floats, using consistent sign and scale conventions.
- **Qualitative input**: nested dictionary of 1–5 scores, where 1 = very strong and 5 = very weak, across business risk, management/governance, and country/structural factors.
- **Optional sovereign input**: sovereign rating (e.g. “BBB”) and outlook (“Positive”, “Stable”, “Negative”) to cap and, if binding, anchor the issuer’s outlook.

The model returns a single `SN_RatingComponents` object that can be logged, serialized, or passed to downstream reporting and visualization tools.

## Execution flow

1. Set weights, financial ratios, qualitative factors, and (optionally) sovereign rating/outlook in the dedicated input section of the notebook.
2. Instantiate `SN_CorporateRatingModel` with chosen quantitative/qualitative weights.
3. Call `model.rate(...)`, which internally:
   - Computes quantitative and qualitative scores,
   - Combines them into a single score,
   - Maps the score to an uncapped rating,
   - Applies sovereign cap if provided,
   - Derives or anchors the outlook,
   - Returns `SN_RatingComponents`.

A sample run is included in the notebook, showing logs for each ratio and factor, intermediate scores, and the final rating/outlook.

## Prototype status and limitations (V1)

- This is an educational and exploratory V1 prototype, not a production or regulatory-grade model.
- Thresholds, weights, and factor definitions are intentionally simple and may differ from agency or internal bank methodologies.
- No calibration to observed default data, stress testing, or back-testing is implemented in this version.
  
## Intended use

- Learning and demonstration of corporate credit rating mechanics in a fully open, inspectable Python implementation.
- Internal prototyping, what-if analysis, and documentation practice around model logic and assumptions.
- A starting point for users who wish to adapt thresholds, add sectors or regions, or plug the engine into data pipelines and dashboards.
