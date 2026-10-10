# 🧹 Olist E-commerce: Data Cleaning & Preparation

A hands-on data cleaning project on the **Olist Brazilian e-commerce dataset** (9 relational tables, ~100K orders). The goal was to turn raw, multi-table data into **clean, analysis-ready tables**, including one **master orders table**, ready for SQL, Power BI or ML work.

> **Scope:** this repository covers the cleaning and preparation stage only. No dashboards or modelling are included here.

---

## 🎯 Objectives

- Audit every table for missing values, duplicates, wrong data types and inconsistent text
- Fix logical inconsistencies between order status and delivery timestamps
- Engineer features analysts actually need (delivery time, freight ratio, review flags, etc.)
- Aggregate item, payment and review data to **one row per order** and join it into a single master table
- Attach coordinates to customers and sellers, recording how precise each match is

---

## 🛠️ Tech Stack

`Python` · `Pandas` · `NumPy` · `Jupyter Notebook`

---

## 📂 Repository Structure

```
├── Olist_Data_Cleaning.ipynb      # Full cleaning workflow
├── Clean Files/
│   ├── Master_Order_Details.csv   # One row per order (orders + items + payments + reviews)
│   ├── Order_Items_Granular.csv   # One row per order item, with cost features
│   ├── Products_Cleaned.csv       # Products with English category names
│   ├── Customers_Cleaned.csv      # Customers with coordinates and geo_match_level
│   └── Sellers_Cleaned.csv        # Sellers with coordinates and geo_match_level
└── README.md
```

> Raw data is not included. Download it from [Kaggle: Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place the CSV files in the same folder as the notebook (keep the original file names; the notebook loads each table **by name**).

---

## 🔄 Cleaning Workflow

| Table | What I did |
|---|---|
| **Customers** | Checked duplicates and nulls, explored states and cities, standardised city names (lowercase, trim, remove accents) |
| **Geolocation** | Cleaned city names, removed points outside Brazil, then collapsed many rows per zip prefix into **one row per zip prefix** (mean latitude/longitude) so later joins don't duplicate rows |
| **Orders** | Converted 5 timestamp columns to datetime; validated missing dates against order status; corrected inconsistent statuses; engineered delivery KPIs |
| **Order Items** | Checked duplicates on (`order_id`, `order_item_id`); added `item_total_cost` and `freight_ratio`; aggregated to order level |
| **Payments** | Aggregated to order level: number of payment methods, methods used, max instalments, total paid |
| **Reviews** | Filled blank comments, created `has_title` / `has_message` flags, keyword flags for delay / quality / praise, review response time; aggregated to order level |
| **Products** | Filled missing categories with `unknown`; imputed missing dimensions with the category median; added `product_volume_cm3`; merged English category names |
| **Sellers** | Checked duplicates, standardised city names, added coordinates |

### Engineered features

| Feature | Meaning |
|---|---|
| `delivery_lead_time_days` | Purchase → delivered to customer |
| `approval_delay_hours` | Purchase → payment approved |
| `seller_processing_hours` | Approval → handed to carrier |
| `carrier_shipping_days` | Carrier pickup → delivered |
| `delivery_delta_days` | Delivered vs. estimated date (negative = early) |
| `item_total_cost`, `freight_ratio` | Price + freight, and freight share of cost |
| `total_order_value`, `total_freight_ratio` | Order-level versions of the above |
| `mentions_delivery_delay`, `mentions_quality_issue`, `mentions_positive_praise` | Keyword flags from review text |
| `order_status_original`, `status_corrected` | Original status and a flag for corrected rows |
| `carrier_date_imputed` | 1 if the carrier date was estimated |
| `is_delivered` | Helper flag for filtering delivered orders |
| `geo_match_level` | How the coordinates were found (`zip`, `city` or `missing`) |

---

## 📍 Geolocation Enrichment (Design Decisions)

Customers and sellers only have a zip-code prefix, so coordinates are added with a **fallback hierarchy**. Every row records how its coordinates were found in `geo_match_level`:

| `geo_match_level` | How the coordinates were found | Precision |
|---|---|---|
| `zip` | Mean lat/lng of all valid geolocation points for that zip-code prefix | Most precise |
| `city` | Mean lat/lng of all valid points in the same city and state (used only when the zip prefix was not found) | Approximate |
| `missing` | No match, so the coordinates are left empty | None |

**Design decisions**

- **No state-average fallback.** A state average can be hundreds of km from the real location and would distort distance or delivery-time analysis, so unmatched rows stay empty instead of being filled with a guess.
- **Brazil-only points.** Coordinates outside Brazil's bounding box are removed before averaging.
- **Same text cleaning everywhere.** One `clean_text()` function (lowercase, trim, remove accents) is applied to the geolocation, customer and seller city names so city-level matching works for names like *São Paulo*.
- **Filtering:** use `geo_match_level == 'zip'` when precise distances are needed.

---

## ⚠️ Assumptions Made

These choices change the data, so they are listed here for transparency:

- Orders marked `delivered` with **no customer delivery date** were re-labelled `shipped` (if a carrier date exists) or `approved` (if not). The original value is kept in `order_status_original`, and corrected rows have `status_corrected = 1`.
- A missing carrier date on a delivered order was **estimated** as *customer delivery date − average carrier transit days*. Imputed rows have `carrier_date_imputed = 1`.
- Missing product dimensions were filled with the **category median** (overall median as a fallback); missing text and photo counts were filled with 0.
- Orders with no payment, item or review records get 0 (or `unknown` for text) in the master table. `avg_review_score` stays empty when there is no review.
- Review keyword flags use a short Portuguese keyword list and are rough indicators, not sentiment analysis.

---

## ▶️ How to Run

```bash
git clone https://github.com/saniyanoori/olist-ecommerce-data-cleaning.git
cd olist-ecommerce-data-cleaning
pip install pandas numpy matplotlib seaborn jupyter
# place the Olist CSV files in the project folder
jupyter notebook Olist_Data_Cleaning.ipynb
```

Cleaned files are written to the `Clean Files/` folder.

---

## 🙏 Data Source

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle, CC BY-NC-SA 4.0).

## 👤 Author

**Saniya Noori** · [LinkedIn](https://www.linkedin.com/in/saniya-noori) · [GitHub](https://github.com/saniyanoori)
