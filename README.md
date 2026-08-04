# Olist Brazilian E-Commerce: Exploratory Data Analysis

Exploratory analysis of **99,441 real orders** placed on Olist, Brazil's largest e-commerce
marketplace, between September 2016 and August 2018. The project answers four business questions:
is fulfilment healthy, which product categories deserve investment, where are the customers, and
does delivery performance affect how customers rate their orders.

Built with pandas, SQL (SQLite in-memory) and matplotlib in a single reproducible notebook.

**Notebook:** [`olist_ecommerce_eda.ipynb`](olist_ecommerce_eda.ipynb) —
[open in Colab](https://colab.research.google.com/github/Raymond-Mendez/olist-ecommerce-eda/blob/main/olist_ecommerce_eda.ipynb)

## Headline findings

| Question | Finding |
|---|---|
| Fulfilment health | 97.0% of orders delivered, 0.6% cancelled |
| Scale | R$15.84M in item revenue across 112,650 items |
| Growth | Grew to a 7,544-order peak in Nov 2017 (Black Friday), then plateaued at ~6,200–7,200 orders/month |
| Top category | health_beauty — R$1.44M revenue at a 4.14 average rating |
| Category to fix | bed_bath_table — 3rd by revenue (R$1.24M), highest volume (11,115 items), weakest rating (3.90) |
| Customer base | 96,096 unique customers; only **3.1% ever order twice** |
| Concentration | São Paulo = 41.9% of customers; SP + RJ + MG = 66.5% |
| Delivery vs rating | 1-star orders waited 20.9 days on average, 5-star orders 10.2 days (Spearman −0.235) |
| **Biggest lever** | Late orders are only **6.7%** of volume but average **2.27 stars vs 4.29** for on-time orders |

### Recommendation

Manage to the *promised* delivery date rather than average transit time. Missing the estimate costs
roughly two stars, and it only happens on 6.7% of orders — a small, targeted operational fix with an
outsized effect on ratings. Because repeat purchasing is just 3.1%, the first order is effectively
the only order, so that first impression carries all of the customer's lifetime value.

## Analysis walkthrough

| Section | What it does | Technique |
|---|---|---|
| 1–2 | Inspect and clean `orders`; convert all five timestamp columns | pandas, `pd.to_datetime` |
| 3 | Order status breakdown with percentage of total | SQL aggregation, subquery |
| 4 | Ten highest-revenue orders | SQL `JOIN` + `GROUP BY` |
| 5 | Translate Portuguese categories to English | pandas `merge`, `value_counts` |
| 6 | Revenue and average review score by category | multi-table merge, `groupby().agg()` |
| 7 | Top 10 states by unique customers | SQL `COUNT(DISTINCT)`, bar chart |
| 8 | Monthly order volume trend | `dt.to_period('M')`, line chart |
| 9 | Delivery days vs review score, plus late-delivery penalty | Spearman/Pearson correlation, `groupby`, bar chart |
| 10 | Executive summary and data quality notes | — |

## Data quality issues found

These are documented in the notebook's closing section because each one changes the answer:

1. **`reviews` is not one row per order** — 99,224 rows but 98,673 unique `order_id` values. Joining
   it directly onto `items` fans out rows and inflates revenue by roughly R$72K. The notebook
   collapses reviews to one score per order and asserts the item row count is unchanged (112,650).
2. **`customer_id` is not a customer** — Olist issues a new `customer_id` per order (99,441) while
   `customer_unique_id` identifies the person (96,096). Using the wrong key overstates the customer
   base by 3.5% and hides the 3.1% repeat rate.
3. **The series is truncated at both ends** — Sep 2016 (4 orders) and Sep–Oct 2018 (16 and 4 orders)
   are partial collection periods, not demand collapses, and are excluded from trend claims.
4. **Missing delivery timestamps** — 2,965 orders lack `order_delivered_customer_date`, mostly
   cancelled or unavailable orders; delivery analysis excludes them.
5. **Revenue definition** — revenue is `price + freight_value` from `items`, which is not the same as
   the `payments` total (that includes instalments and vouchers).

## Dataset

Seven related tables, loaded directly from public S3 storage — no manual download needed.

| File | Rows |
|---|---|
| `olist_orders_dataset.csv` | 99,441 |
| `olist_order_items_dataset.csv` | 112,650 |
| `olist_customers_dataset.csv` | 99,441 |
| `olist_products_dataset.csv` | 32,951 |
| `olist_order_reviews_dataset.csv` | 99,224 |
| `olist_order_payments_dataset.csv` | 103,886 |
| `product_category_name_translation.csv` | 71 |

Source: the [Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce),
released by Olist and hosted here via the TripleTen curriculum S3 bucket. Currency is Brazilian real (R$).

## Running it

```bash
pip install -r requirements.txt
jupyter notebook olist_ecommerce_eda.ipynb
```

Then Restart & Run All — every cell executes top to bottom in order with no local data files.

## Tech stack

Python · pandas · SQLite (SQL via `pd.read_sql`) · matplotlib · Jupyter

## License

[MIT](LICENSE)
