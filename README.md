# Banner Funnel Performance Analysis

A funnel analysis in Python comparing five product banner categories — clothes, sneakers, accessories, sports_nutrition, and company — across the impression → click → order funnel, to determine which banner performs best and whether the differences are statistically significant.

## Overview

This project analyzes an 8,471,220-row event log to evaluate banner performance across three funnel stages: `banner_show` (impression), `banner_click`, and `order`. Beyond simply ranking banners by conversion rate, the analysis validates data integrity first, tests whether observed differences are statistically significant (not just numerically different), checks whether the winning banner holds up across device segments, and translates the result into an estimated business impact.

## Tech Stack

- **Language:** Python
- **Libraries:** pandas, numpy, matplotlib, statsmodels (proportions_ztest, proportion_effectsize)

## Methodology

The analysis follows nine sections:

1. **Load & Inspect** — Shape, unique order/user/page counts, null audit, and a categorical inventory of products, site versions (device), and event titles.
2. **Data Quality & Integrity Checks** — Confirms `target=1` only ever appears on `order` events (never on shows or clicks), and checks for duplicate `page_id` rows that would indicate double-counted events.
3. **Funnel Construction** — Aggregates raw event counts into per-product shows, clicks, and orders, aligned on a common product index so any product with zero events in a stage still appears as 0 rather than being silently dropped.
4. **Primary Metrics** — Computes CTR (click/show), click-to-order conversion, and overall conversion (order/show) per product, then ranks products by overall conversion to identify the best and weakest performers.
5. **Statistical Significance Testing** — Runs a two-proportion Z-test with Cohen's h effect size, comparing the top-converting banner against every other banner on both overall conversion and CTR — separating "is the top banner different" from "is the top banner different by a large margin."
6. **Segment Analysis — Device** — Rebuilds the full funnel separately for mobile and desktop, then Z-tests mobile vs. desktop conversion per product to check whether the overall winner holds consistently across device types.
7. **Funnel Visualization** — Bar charts of CTR, click-to-order conversion, and overall conversion by banner.
8. **Business Impact Framing** — Estimates incremental orders if every product matched the best performer's conversion rate, applied against total observed impression volume.
9. **Final Summary** — Consolidates funnel leaders, significance results, device segment findings, and a prioritization recommendation.

## Key Design Decisions

- **Integrity checks precede metric calculation.** A `target=1` flag appearing outside `order` events, or duplicate `page_id` rows, would silently distort every downstream conversion rate — so these are caught first, before any funnel math runs.
- **Products are reindexed against a common list** before computing shows/clicks/orders, so a product with zero recorded orders shows up explicitly as `0` rather than disappearing from the analysis — this is what surfaces banners with no recorded orders at all, a materially different problem from "low conversion."
- **Significance testing pairs every comparison with Cohen's h**, not just a p-value, since at 8M+ rows even tiny, practically meaningless differences can register as statistically significant — effect size is what separates "real" from "large enough to act on."
- **Device segmentation re-tests the winner from scratch** rather than assuming the aggregate result generalizes — a banner that wins overall could still underperform on one device, which would change how creative/budget should actually be allocated.
- **Business impact is explicitly framed as directional**, noting in the code that it assumes impression volume stays constant across creative changes and ignores margin/price differences between product categories — this caveat is deliberately preserved in the output rather than left implicit.

## Key Findings

*(Fill in with actual run output — the script prints all of the below dynamically rather than hardcoding results.)*

- **Best-converting banner (overall, impression → order):** `[best_product]` at `[X.XX]%`
- **Weakest-converting banner:** `[worst_product]` at `[X.XX]%`
- **Banners with zero recorded orders:** `[list, if any]` — these can't be significance-tested and represent a separate creative/tracking issue worth investigating on their own.
- **Statistical significance:** `[best_product]` vs. each other banner — Z-stat, p-value, and Cohen's h per comparison (see Section 5 output).
- **Device consistency:** whether the top banner's conversion advantage holds on both mobile and desktop, or is concentrated in one device type (see Section 6 output).
- **Business impact:** estimated incremental orders if all products matched the top banner's conversion rate, given total observed impression volume (see Section 8 output).

## How to Run

1. Install dependencies: `pip install pandas numpy matplotlib statsmodels`
2. Update the file path in Section 1 to point to your local copy of `product.csv`.
3. Run the script top to bottom — later sections (funnel metrics, significance tests, device segmentation) depend on the `shows`/`clicks`/`orders` series built in Section 3.
4. Three matplotlib charts (CTR, click-to-order conversion, overall conversion by banner) will render in Section 7.

## Notes

- With 8.4M rows, even very small conversion differences can be statistically significant — Cohen's h is reported alongside every p-value specifically to guard against over-reacting to a real-but-trivial effect.
- The business impact estimate in Section 8 is explicitly a directional projection, not a revenue forecast — it does not account for margin differences between product categories or the possibility that show volume itself would shift under a different banner mix.
