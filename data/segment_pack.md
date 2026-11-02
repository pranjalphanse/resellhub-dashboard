# ResellHub — Reseller Segment Pack (for Part 3)

*Prepared by the ResellHub data-science team. You are **not** expected to build the model. Inputs: for each of the 56 resellers who received at least one order (file `reseller_summary.csv`): `n_orders` and `avg_value` (average order value in ₹), both standardised, then clustered with K-means.*

| k | Within-cluster sum of squares (elbow curve) | Silhouette score |
|---|---|---|
| 2 | 62.41 | 0.530 |
| 3 | 29.29 | 0.581 |
| 4 | 16.93 | 0.569 |
| 5 | 12.19 | 0.496 |

**Cluster profile at k = 3** (cluster IDs are arbitrary labels):

| Cluster ID | Resellers | Average orders per reseller | Average order value (₹) | Return rate | Margin % | Margin per reseller (₹) |
|---|---|---|---|---|---|---|
| 0 | 35 | 7.2 | 535.3 | 17.7% | 9.8% | 365.1 |
| 2 | 8 | 35.4 | 542.3 | 13.7% | 9.7% | 1760.5 |
| 1 | 13 | 5.0 | 1426.2 | 6.2% | 14.0% | 1016.2 |
