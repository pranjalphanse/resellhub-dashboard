# ResellHub Platform Capstone

> **"Is ResellHub's growth self-reinforcing, and which reseller segment and region should get the onboarding subsidy?"**

---
## Team Name
ResellVision

## Team Members

| Name | Contribution |
|------|-------------|
| Pranjal (GR) | Parts 3: workings: sheets Monthly, Network, Segments, platform analysis |
| Amrutha | Parts 1–4: Data preparation, Power BI dashboard, segmentation, recommendation |
| Darshini | Parts 1–4: Data preparation, Power BI dashboard, segmentation, recommendation |
| Surya | Parts 4: recommendation, presentation |
| Vishesh | Parts 4: recommendation, presentation |

---

## Repository Contents

| File | Description |
|------|-------------|
| `README.md` | This file — team info, written answers, and recommendation summary |
| `data/orders.csv` | 600 marketplace orders, Jan–Jun 2026 (supplied, unmodified) |
| `data/resellers.csv` | 60 resellers with region and join date (supplied, unmodified) |
| `data/segment_pack.md` | K-means segmentation from the data-science team (supplied, unmodified) |
| `resellhub_analysis.xlsx` | Part 3 workings: sheets Monthly, Network, Segments |
| `resellhub_dashboard.pbix` | Parts 1–2: Power BI dashboard with star schema, measures, and visuals |
| `ResellHub_Presentation.pptx` | Part 4: 6–8 slide presentation deck |

---

## Part 1 — Data Preparation (Written Answers)

### 1.1 OLTP vs OLAP Classification

| Statement | Classification | Reason |
|-----------|---------------|--------|
| (a) Records every order the moment it is placed | **OLTP** | Transactional write at event time — hallmark of operational systems |
| (b) Compares monthly sales by region over several years | **OLAP** | Multi-dimensional historical aggregation — classic analytical workload |
| (c) Stores each order's current status and overwrites it on update | **OLTP** | Mutable current-state record — overwrites are OLTP, forbidden in warehouses |
| (d) Keeps historical snapshots and never overwrites, only marks old rows expired | **OLAP** | Non-volatile + time-variant: warehouse appends, never overwrites |
| (e) Feeds the executive dashboard | **OLAP** | Dashboards read from warehouses, not live OLTP systems |
| (f) Optimised for many small, fast writes | **OLTP** | Row-store, write-optimised engine (MySQL/Postgres) — pure OLTP workload |

### 1.2 Four Properties of a Data Warehouse

| Property | Definition | ResellHub Example |
|----------|-----------|-------------------|
| **Subject-Oriented** | Organised around business subjects, not application functions | Warehouse centres on Orders and Resellers — every analysis derives from these two subjects |
| **Integrated** | Data from disparate sources cleaned, standardised, and unified | `orders.csv` + `resellers.csv` merged on `reseller_id`; region normalised from 13 raw spellings to 4 clean values |
| **Non-Volatile** | Data is loaded and read only — never updated or deleted | January orders remain in the warehouse when June data arrives; nothing is overwritten |
| **Time-Variant** | Every record carries a time dimension for historical comparison | `order_date` enables Jan–Jun monthly trend analysis (orders grew 60 → 140 over six months) |

---

## Part 2 — Model and Dashboard (Written Answers)

### 2.1 Star Schema

- **Fact table:** `orders` (600 rows) — transactional grain, one row per order
- **Dimension table:** `resellers` (60 rows) — descriptive attributes (name, region, join date)
- **Relationship:** `orders[reseller_id]` → `resellers[reseller_id]`, many-to-one, active

**Snowflake extension:** A snowflake schema would split `region` into a separate `dim_region` table with its own `region_id` as primary key, linked from `resellers[region_id]` as a foreign key. This adds a second level of normalisation beyond the star.

### 2.5 Dashboard Business Questions

**Q1 — Which region has the highest order value?**
East — ₹1,02,620.

**Q2 — Which category has the highest order value?**
Beauty — ₹1,00,460.

**Q3 — Who are the top 5 resellers by order value?**
1. Farida Reddy (R041) — ₹27,660
2. Girish Pillai (R031) — ₹26,210
3. Manoj Menon (R035) — ₹23,480
4. Hema Nair (R012) — ₹21,450
5. Manoj Patel (R040) — ₹17,510

**Q4 — Is overall Margin % at or above the 10% target, and which region is below?**
Yes — overall Margin % is 10.9%, above the 10% target. The only region below the target is **North at 9.1%**. The matrix shows North in red; all others are green (East 11.0%, South 11.8%, West 11.8%).

---

## Part 3 — Platform Health and Reseller Segments (Written Answers)

### 3.2 Network Effects

**Two sides of the platform:**
- **Supply side:** Resellers — list products, fulfil orders, generate variety
- **Demand side:** Buyers — browse, purchase, generate revenue

**Same-side network effect (Reseller ↔ Reseller):** More resellers on the platform share supplier relationships, pricing benchmarks, and reputation signals — each reseller's listing quality improves as the community grows.

**Cross-side network effect (Reseller ↔ Buyer):** More resellers → greater product variety → more buyers attracted to the platform. Conversely, more buyers → higher order volumes → more resellers want to join for the revenue opportunity.

**Correlation:** Active resellers vs active buyers across the six months: **r = 0.984** — extremely strong positive correlation, consistent with a cross-side network effect. However, six data points sharing a common growth trend are not proof of cause. Marketing spend, seasonality, or a third confounding variable could explain both sides growing simultaneously without a direct causal mechanism.

### 3.3 Metcalfe's Law

| Month | n (Resellers + Buyers) | n² (Network Value) |
|-------|----------------------|-------------------|
| January | 38 | 1,444 |
| February | 50 | 2,500 |
| March | 72 | 5,184 |
| April | 101 | 10,201 |
| May | 119 | 14,161 |
| June | 141 | 19,881 |

- n grew **3.71×** (141 ÷ 38)
- Network value grew **13.8×** (19,881 ÷ 1,444) — the square of 3.71×

**Winner-take-all implication:** A rival growing at half the pace (n ≈ 71 by June) would have only n² ≈ 5,041 — roughly one-quarter of ResellHub's network value. The faster-growing platform's advantage compounds exponentially, not linearly, making it progressively harder for a slower rival to catch up.

### 3.4 Segment Profiles (k = 3)

**Choosing k:** Silhouette score is highest at k = 3 (0.581). The elbow curve bends around k = 3–4 (within-cluster sum of squares drops from 62.4 to 29.3 — the largest single drop — then to 16.9). Both methods agree on k = 3.

| Cluster Name | ID | Resellers | Avg Orders | Avg Value (₹) | Return Rate | Margin % | Margin/Reseller (₹) |
|---|---|---|---|---|---|---|---|
| **High-Volume Bulk Sellers** | 2 | 8 | 35.4 | 542.3 | 13.7% | 9.7% | 1,760.5 |
| **Premium Niche Sellers** | 1 | 13 | 5.0 | 1,426.2 | 6.2% | 14.0% | 1,016.2 |
| **Casual Low-Margin Sellers** | 0 | 35 | 7.2 | 535.3 | 17.7% | 9.8% | 365.1 |

### 3.5 Region Supply

| Region | Resellers | Orders | Orders per Reseller | Status |
|--------|-----------|--------|-------------------|--------|
| North | 15 | 216 | **14.4** | Most under-supplied |
| East | 15 | 152 | 10.1 | Under-supplied |
| West | 15 | 128 | 8.5 | Balanced |
| South | 15 | 104 | 7.0 | Well-supplied |

The most under-supplied region is **North** (highest orders per reseller = 14.4, nearly double South's 7.0).

---

## Part 4 — Recommendation (Written Answers)

### 4.1 Three Platform Strategic Moves

| Move | Platform Idea | Supporting Number |
|------|--------------|-------------------|
| **1. Build critical mass of resellers in North** | Critical Mass — minimum user base at which network effects become self-sustaining | North: 14.4 orders/reseller (2× South's 7.0); North margin 9.1% — only region below 10% target |
| **2. Lock in high-volume sellers with complementary services** | Complementary Services & Switching Costs — extras that make leaving costly | Cluster 2: ₹1,760.5 margin/reseller; only 8 such sellers — each is irreplaceable |
| **3. Use platform data to personalise buyer experience** | Data as a Moat — proprietary behavioural data compounds and cannot be replicated | 149 active buyers; Metcalfe ratio 13.8×; Beauty ₹1,00,460 is top category |

### 4.2 Subsidy Decision

**Recommended segment:** High-Volume Bulk Sellers (Cluster 2)
**Recommended region:** North

**Rationale:**
- Cluster 2 delivers ₹1,760.5 margin per reseller — 4.8× more than Cluster 0's ₹365
- Only 8 such sellers exist — thin supply of these high performers
- North has 14.4 orders per reseller — the highest demand-supply gap
- North margin is 9.1% — the only region below the 10% target
- Adding Cluster 2 resellers in North addresses both the margin shortfall and the supply gap

**Risk 1 — Subsidy attracts low-margin sellers mimicking Cluster 2:**
New entrants may post high order counts via discount pricing without delivering expected margin. **Monitor:** Margin per reseller at 30/60/90 days post-onboarding; flag if below ₹800 (45% of Cluster 2 average). **Mitigate:** Gate full subsidy payout behind a 60-day trial: release only if orders ≥ 20, return rate < 15%, and margin/reseller > ₹800.

**Risk 2 — Correlation ≠ Causation (growth may be seasonal, not network-driven):**
If Jan–Jun growth reflects festive-season peaks rather than network effects, adding North resellers may not trigger the expected buyer-side growth. **Monitor:** North-specific active buyers per active reseller each month; compare to South/West as control regions. **Mitigate:** Run a 3-month controlled pilot: subsidise 4–5 North sellers; compare North buyer acquisition rate vs South (no-subsidy control); gate full rollout on Month 3 results.

---

## 5-Sentence Recommendation Summary

ResellHub's growth is self-reinforcing: active resellers and active buyers correlate at r = 0.984, and Metcalfe's n² network value grew 13.8× while total participants grew only 3.7×, creating a winner-take-all dynamic against slower rivals. The K-means segmentation at k = 3 (silhouette 0.581) reveals that the 8 High-Volume Bulk Sellers (Cluster 2) generate ₹1,760.5 margin per reseller — 4.8× higher than the 35 Casual Low-Margin Sellers — making them the highest-ROI target for the onboarding subsidy. The North region is the most under-supplied with 14.4 orders per reseller (double South's 7.0) and the only region below the 10% margin target at 9.1%, so concentrating the subsidy there addresses both the supply gap and the margin shortfall. The primary risks are that subsidised sellers may mimic Cluster 2's volume without its margin, and that the observed correlation may reflect seasonal growth rather than a true network effect. Both risks are managed through a gated 60-day payout, performance thresholds (orders ≥ 20, returns < 15%, margin > ₹800), and a controlled North-vs-South pilot before full rollout.

---

## Tools Used

- **Microsoft Excel** — Part 3 workings (pivot tables, formulas, correlation)
- **Power BI Desktop** — Parts 1–2 (data preparation via Power Query, star schema model, DAX measures, dashboard visuals)
- **PowerPoint / Google Slides** — Part 4 presentation
- **GitHub** — Repository hosting and submission
