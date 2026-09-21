# Why don't Olist customers come back?

Only 3% of customers on Olist (a Brazilian e-commerce marketplace) ever place a second order. This project digs into why, tests the most popular explanation — and finds out it's wrong.

The notebook itself is written in Japanese because I'm using this portfolio for job hunting in Japan. This README covers the full analysis in English.

## Background

Olist connects small Brazilian sellers to large marketplaces. The public dataset on Kaggle covers ~100k orders from 2016 to 2018 across 9 relational tables (orders, customers, order items, payments, reviews, products, sellers, geolocation, category translations).

Acquiring new customers is expensive, so a 3% repeat rate is a real business problem. The question I set out to answer: **what is actually blocking repeat purchases, and where should Olist spend money first?**

My starting hypotheses:

1. Late deliveries tank review scores, and unhappy customers don't come back
2. Repeat rates differ a lot by product category
3. A small group of high-value customers behaves differently from everyone else (RFM)

## What I found

**1. The repeat rate really is 3.0%** — 93,358 delivered-order customers, and only ~2,800 of them ever bought twice. Monthly order volume roughly tripled year over year (avg 2,435 orders/month in H1 2017 to 6,864 in H1 2018), but all of that growth came from new customers.

**2. Late delivery destroys reviews, but reviews don't predict repurchase.** This was the surprise. Delivery delay has a brutal effect on satisfaction: orders delivered more than a week late average 1.69 stars vs 4.28 for early ones, and 70% of them get 1 star. So the first half of hypothesis 1 holds. But the second half fails completely: customers who left a 1-star review repurchase at 2.97%, and 5-star customers at 3.00% (chi-square p = 0.875 — no difference at all). Even a delayed first order only drops repurchase from 3.04% to 2.47%. Statistically significant (p = 0.012), but 0.6 points is not a business case.

In other words, the common advice "fix delivery and retention will follow" doesn't hold here. Even happy customers leave. The 3% is structural, not a service failure.

**3. What actually separates repeaters is the product category.** Home appliances repeat at 10.5%, fashion accessories at 8.8%, furniture at 7.2% — versus 2.9% for electronics. That's a 3.5x spread. And the RFM segmentation shows revenue is concentrated in one-time buyers: customers who bought once, recently, and spent a lot ("promising one-timers") are 24% of customers but 38% of revenue. All repeaters combined are just 5.6% of revenue.

The cohort analysis confirms the pattern is structural: for every monthly cohort from Jan 2017 onward, next-month repurchase sits between 0.3% and 0.7%. It never improved as the company grew.

## Recommendations

Ordered by expected impact:

1. **Post-purchase CRM aimed at the ~22,500 "promising one-timers"**, prioritizing buyers in high-repeat categories (appliances, fashion accessories, furniture, bed & bath). A +3pt lift in their repeat rate is worth roughly R$110k/year at the observed R$161 average order value. This is the only lever the data actually supports for retention.
2. **Keep fixing late deliveries — but budget it as brand protection, not retention.** 32% of 1-2 star first orders were late. The direct repeat-revenue impact of halving delays is tiny (~R$3k), so the justification is marketplace trust and review scores, and the ROI should be judged on that basis.
3. **If retention stays structurally low, optimize one-timer value instead**: cross-sell and basket size fit Olist's marketplace model better than loyalty mechanics.

## How it's done

Everything is in [`olist_analysis.ipynb`](olist_analysis.ipynb). The analysis was designed before touching the data — the full design doc (business problem → objective → hypotheses → verification plan) is in [`analysis_design_ja.md`](analysis_design_ja.md) (Japanese). Techniques used, roughly in order:

- **Joins across the relational schema** (orders ← customers ← items ← products ← reviews ← payments). One trap worth knowing: `customer_id` changes with every order in this dataset. Counting customers requires `customer_unique_id`, otherwise the repeat rate comes out as ~0%.
- **Repeat-rate calculation** on delivered orders only, with duplicate reviews per order deduplicated (keeping the lowest score, the conservative choice).
- **Hypothesis testing with chi-square tests** (`scipy.stats`) for review score vs repurchase and delay vs repurchase, rather than eyeballing bar charts.
- **First-order attribution**: repurchase behavior is linked to each customer's *first* delivered order, so the delay/review of that order can be treated as the potential cause.
- **RFM segmentation** with one adjustment: since 97% of customers have frequency = 1, quartile-based F scores are meaningless, so F is binary (repeater or not) while R and M use quartiles.
- **Cohort retention analysis** by first-purchase month, plotted as a heatmap.
- **Impact sizing** for each recommendation (customers × rate lift × average order value), so the proposals come with numbers attached.

Every chart carries a one-line takeaway in its title — a habit I picked up from the Japanese analytics book 『伝わる分析』.

## Running it

```
pip install pandas matplotlib scipy
```

Download the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) from Kaggle, unzip the CSVs into a `data/` folder next to the notebook, and run all cells. The data isn't committed to this repo (see `.gitignore`) per Kaggle's distribution terms.

## Limitations

- The data ends in October 2018, so customers who first bought near the end had less time to repurchase. The true repeat rate is slightly understated.
- Review → repurchase is tested as an association on first orders, not a causal estimate. Propensity score matching would be the natural next step.
- Geolocation went unused; delivery-time analysis by state could sharpen recommendation 2.

## Stack

Python (pandas, numpy, matplotlib, scipy), Jupyter. A Streamlit dashboard version is planned.
