# Fashion Sales — Dataset Guide and Analysis

This document does two things:

1. **Explains the dataset** to someone who has never seen it (Part A).
2. **Reports the full analysis** — seasonality, year-over-year change, temperature effects, discounting, advertising, returns and ratings (Part B).

All numbers come from `fashion_sales.csv` (240 rows).

---

# Part A — What this dataset is

## A.1 What it covers

It is a **two-year sales summary for a fashion retailer**, covering **January 2024 through December 2025**. Rather than listing individual transactions, every row is an **aggregate**: sales for one *product category*, in one *sales channel*, in one *month*.

- **5 product categories:** Accessories, Denim, Dresses, Footwear, Outerwear
- **2 sales channels:** Online, Store
- **24 months:** 2024-01 … 2025-12
- **5 × 2 × 24 = 240 rows** — a complete, balanced grid with no missing combinations and no duplicates.

The dataset is small but tidy: it is designed for trend and comparison analysis (month-over-month, category-vs-category, channel-vs-channel), not for order-level work.

## A.2 How it is organized

Each row is identified by three keys — `month`, `category`, `channel` — and carries the measurements for that cell.

| Column | Type | Meaning | Notes |
|---|---|---|---|
| `month` | text | `YYYY-MM` label | 24 values, 2024-01 … 2025-12 |
| `year` | integer | 2024 or 2025 | derived from `month` |
| `month_num` | integer | 1–12 | derived from `month` |
| `category` | text | Product category | 5 values |
| `channel` | text | Online or Store | 2 values |
| `avg_temp_F` | decimal | Average temperature that month (°F) | **one value per month**, repeated on every row of that month |
| `list_price` | integer | Full retail price | **fixed per category** |
| `discount_pct` | integer | Discount applied | one of 0, 10, 15, 20, 25, 30, 40 |
| `sale_price` | decimal | Price actually paid per unit | see dependencies below |
| `ad_spend` | integer | Advertising spend for that month/category/channel | 2,050–19,270 |
| `avg_rating` | decimal | Average product rating | 3.4–5.0 |
| `units_sold` | integer | Units sold | 220–2,488 |
| `revenue` | decimal | Revenue for the row | = `sale_price × units_sold` |
| `return_rate` | decimal | Fraction of units returned | 0.01–0.36 |

**Key structural facts to keep in mind:**

- `month`, `year` and `month_num` are three views of the same thing (e.g. `2024-01` = year 2024, month 1). They always agree.
- `list_price` and `avg_temp_F` are **not row-specific measurements** — they are fixed for their category (price) or their month (temperature) and copied across rows. Treating them as 240 independent observations would overstate how much information they carry.
- `sale_price`, `units_sold`, `revenue`, `ad_spend`, `avg_rating` and `return_rate` are genuine per-row measurements.

## A.3 Which columns depend on each other

Three relationships matter:

1. **`revenue` is exactly derived from `sale_price` and `units_sold`.** For every one of the 240 rows, `revenue = sale_price × units_sold` holds to floating-point precision. This is a calculated column — it carries no independent information.

2. **`sale_price` is *approximately* derived from `list_price` and `discount_pct` — but not exactly.**
   The intended formula is `sale_price = list_price × (1 − discount_pct/100)`, and the data follows it only loosely: the actual value sits within roughly **±10%** of that formula (ratio range 0.90–1.10, mean 1.00). So `sale_price` behaves like the discounted price **plus random noise**.

3. **`list_price` is fixed per category**, so `list_price` is really a category attribute:

   | Category | List price |
   |---|---:|
   | Accessories | $35 |
   | Denim | $70 |
   | Dresses | $85 |
   | Footwear | $110 |
   | Outerwear | $180 |

Because of (1) and (2): **`revenue` is ultimately a noisy function of category (`list_price`), `discount_pct` and `units_sold`.** The one genuinely independent driver of revenue is `units_sold`.

## A.4 Surprises and caveats

- **21 rows have `sale_price` greater than `list_price`** — a "sale" price above the full price, which is impossible in reality. It happens even at 0% discount (e.g. Outerwear 2024-09: list $180, discount 0, sale $197.18). This is the visible symptom of the ±10% noise.
- **None of the 43 rows with `discount_pct = 0` have `sale_price` exactly equal to `list_price`** — again the noise.
- **`revenue` is not a pure measurement** — it is `sale_price × units_sold`, so it inherits the noise above. Treat revenue figures as approximate.
- **`avg_temp_F` is a month-level attribute**, not a per-row one. It varies by month (31.3–80.4 °F) but is identical across all 10 rows of a given month, and it is the same value for both channels and all categories.
- **`return_rate` is small in absolute terms but large in effect** — it peaks at 36%, and roughly one in six units comes back overall.
- The data is **fully balanced and complete**: no missing values, no negative values, no duplicate keys. Any oddity in the data is about *consistency* (the price noise), not *completeness*.

---

# Part B — Analysis

## B.1 Headline totals

| Metric | Value |
|---|---:|
| Rows | 240 |
| Total revenue | $13,220,482 |
| Total units sold | 202,500 |
| Total ad spend | $2,337,530 |
| Overall revenue ÷ ad spend | 5.66 |
| Overall unit-weighted return rate | 17.6% |
| Mean rating | 4.22 |

## B.2 Seasonality

Revenue and units concentrate at year-end:

| Window | Share of units | Share of revenue |
|---|---:|---:|
| Nov + Dec (2 of 24 months) | 22.6% | 21.9% |
| Oct–Dec (a quarter of the year) | 30.1% | 29.8% |

- Peak month: **2025-12** — 12,778 units, $754,943.
- Trough month: **2024-02** — 6,290 units, $478,727. That is a **2.0×** unit swing.
- A second, category-specific spike occurs in **July**, driven almost entirely by **Dresses** (Dresses average ~4,320 units in July versus ~1,990 in a typical month — **2.2×**). July is 10.3% of annual units.
- Outerwear is the most extreme: its calendar-month average ranges from **2,486 units (Dec)** down to **624 (Jun)** — a 4× swing.

Overall, monthly temperature correlates only weakly with total units (r = −0.16) and total revenue (r = −0.39), because winter goods and summer dresses move in opposite directions and partly cancel.

## B.3 2024 vs 2025

| Metric | 2024 | 2025 | Change |
|---|---:|---:|---:|
| Revenue | $6,406,734 | $6,813,748 | **+6.4%** |
| Units | 97,712 | 104,788 | **+7.2%** |
| Ad spend | $1,098,180 | $1,239,350 | **+12.9%** |
| Revenue ÷ ad spend | 5.83 | 5.50 | **−5.8%** |
| Unit-weighted return rate | 17.2% | 17.9% | +0.7 pt |
| Mean rating | 4.24 | 4.20 | −0.04 |
| Average discount | 22.9% | 22.3% | −0.6 pt |
| Average sale price | $73.54 | $73.87 | +0.4% |

Every category grew, led by Dresses (+8.7%) and slowest Footwear (+5.1%). The key point: **revenue and units grew ~6–7%, but advertising grew 12.9% — roughly twice as fast — so marketing efficiency fell.** Growth in 2025 was bought with extra spend, while returns rose slightly and ratings dipped.

## B.4 Temperature effects by category

Correlation of monthly temperature with monthly units (n = 24):

| Category | r(units vs temp) | Slope (units per °F) | r(revenue vs temp) |
|---|---:|---:|---:|
| Outerwear | **−0.82** | −32.6 | **−0.89** |
| Dresses | **+0.68** | +37.2 | **+0.85** |
| Denim | −0.35 | −8.7 | −0.30 |
| Accessories | −0.30 | −7.9 | −0.34 |
| Footwear | −0.26 | −6.4 | −0.07 |

- **Outerwear is strongly cold-weather** (r = −0.82): each additional 1 °F costs about 33 units/month; revenue correlation −0.89.
- **Dresses are the mirror image** (r = +0.68): each additional 1 °F adds about 37 units/month; revenue +0.85.
- Footwear, Denim and Accessories lean slightly negative but weakly; their seasonality is far milder.

## B.5 Do bigger discounts sell more units? — Yes

| Discount | n rows | Mean units | Median units |
|---:|---:|---:|---:|
| 0% | 43 | 586 | 577 |
| 10% | 12 | 607 | 562 |
| 15% | 13 | 812 | 773 |
| 20% | 45 | 784 | 755 |
| 25% | 28 | 802 | 768 |
| 30% | 46 | 911 | 917 |
| 40% | 53 | **1,129** | 1,124 |

- Overall **r(discount, units) = 0.51** (Spearman 0.54).
- Within each category the link is **stronger** (0.55–0.75) because that removes the category/price confound: Denim 0.75, Footwear 0.69, Outerwear 0.57, Dresses 0.55, Accessories 0.55.
- Pooled within-category regression: **+13.8 units per +1 point of discount**, i.e. **+10 points ≈ +138 units per row** (~+24% versus the 586-unit, 0%-discount base).
- The payoff is partly given back on price: **r(discount, revenue) = only 0.20**, and **r(discount, sale_price) = −0.27**. Volume rises, price falls, net revenue rises only modestly.
- Caution: units also correlate **−0.35 with `list_price`**, so cheaper categories sell more units — some of the raw discount effect is category mix.

## B.6 Does advertising pay off? — Weakly, with diminishing returns

- Overall **r(ad_spend, revenue) = 0.21** and **r(ad_spend, units) = 0.25** — weak.
- **ROAS overall = 5.66** ($5.66 revenue per $1 of ad spend). By channel: **Store 6.27** versus **Online 5.20**.
- Ad-spend quartiles show heavy diminishing returns:

| Ad-spend quartile | Mean ad spend | Mean revenue | Mean units |
|---|---:|---:|---:|
| Q1 | $3,996 | $49,150 | 773 |
| Q2 | $7,738 | $50,519 | 796 |
| Q3 | $11,225 | $57,462 | 791 |
| Q4 | $16,000 | $63,211 | 1,016 |

Quadrupling spend (Q1 → Q4) buys only **+29% revenue / +31% units** — far below proportional.
- Within category the link is stronger (Footwear 0.61, Denim 0.56, Accessories 0.38, Dresses 0.37, Outerwear 0.18); the weak overall figure is partly a category-mix effect.
- In the **Store** channel there is essentially **no relationship** (r = 0.04) versus Online (0.29) — store revenue appears driven by footfall rather than ad spend.

## B.7 Returns — a channel problem, not a discount problem

- Overall **17.6%** unit-weighted; **35,642 units returned**, implying **~$2.37M (18.0%) of gross revenue at risk**.
- **Online 24.6% versus Store 9.9%** — online returns are **2.5×** the store rate, and this holds in every category:

| Category | Online | Store |
|---|---:|---:|
| Footwear | 29.7% | 14.7% |
| Dresses | 28.9% | 14.9% |
| Denim | 22.3% | 6.6% |
| Accessories | 21.2% | 6.5% |
| Outerwear | 21.1% | 7.0% |

- By category overall: Dresses and Footwear worst (both **22.4%**); Outerwear 14.4%, Denim 14.7%, Accessories 14.1%.
- **Discounting does not drive returns:** r(return, discount) = −0.10, and the unit-weighted return rate by discount level is flat-to-slightly-down (19.2% at 0% → 17.4% at 40%).
- **Ad spend looks correlated with returns overall (r = +0.23), but this is a channel artifact:** within Online it is −0.13 and within Store −0.01. The highest-spend quartile is 76.7% online (return 21.1%) versus the lowest-spend quartile at 36.7% online (16.1%) — it is the channel, not the spend.
- Return versus sale price: r = 0.05 — negligible.

## B.8 Ratings — high and stable; only returns move them

- Mean **4.219**, median 4.2, range 3.4–5.0.
- Channel: **Store 4.29** versus **Online 4.15** (mirrors the returns gap).
- Category spread is tiny: Accessories 4.25 (best) to Denim 4.18 (worst) — a 0.07 range.
- **Ratings are unrelated to price and discount** (r ≈ −0.01 for both).
- The one real relationship is with returns: **r(rating, return) = −0.35**.

| Return quartile | Mean return rate | Mean rating |
|---|---:|---:|
| Q1 | 6.1% | 4.34 |
| Q2 | 13.7% | 4.26 |
| Q3 | 21.0% | 4.16 |
| Q4 | 28.5% | 4.11 |

Lower-rated products come back more, which points to fit/quality issues rather than price.

## B.9 Correlation matrix (row level, n = 240) — key pairs

| Relationship | r | Reading |
|---|---:|---|
| list_price ↔ sale_price | 0.91 | as expected |
| units_sold ↔ sale_price | **−0.52** | cheaper items sell more units |
| units_sold ↔ list_price | −0.35 | same, at category level |
| revenue ↔ list_price | **0.67** | price drives revenue more than volume does |
| discount ↔ units_sold | **0.51** | discounting lifts volume |
| ad_spend ↔ units_sold / revenue | 0.25 / 0.21 | weak |
| return_rate ↔ avg_rating | **−0.35** | unhappy customers return more |
| ad_spend ↔ return_rate | 0.23 | channel confound (see B.7) |

## B.10 Bottom line

1. **Seasonality is real and weather-driven.** About 30% of annual revenue lands in Oct–Dec, plus a Dresses-led July spike. Outerwear and Dresses respond to temperature in near-opposite ways (−0.82 vs +0.68), so plan inventory by weather, not by a single "season."
2. **2025 grew ~6–7% but marketing efficiency fell** (ROAS 5.83 → 5.50, ad spend +12.9%), with returns up and ratings down slightly.
3. **Discounts genuinely move volume** (+13.8 units per point; +10 points ≈ +138 units/row) but only modestly lift revenue (a −0.27 price offset), and they do **not** increase returns.
4. **Advertising has weak, diminishing returns** (quadrupling spend buys +29% revenue; the store channel shows no effect at all).
5. **Returns are the biggest controllable problem, and it is online-specific** (24.6% vs 9.9%, 2.5×), worst in Dresses and Footwear — and returns are what depress ratings, not price or discounting.

## B.11 Caveats

- `sale_price` carries ~±10% noise and 21 rows exceed `list_price`, so `revenue` (a product of `sale_price` and `units_sold`) is approximate.
- Temperature correlations use 24 monthly aggregates, so they are strong but based on a small sample.
- `avg_temp_F` and `list_price` are month- and category-level attributes repeated across rows; they are not independent observations.
- This is aggregate data — no customer-, order- or product-level detail is available, so causal claims (e.g. "ads caused revenue") cannot be established from it alone.
