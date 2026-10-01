# 🛒 Zepto Inventory & Pricing Analysis (SQL)

An end-to-end SQL project that explores, cleans and analyses a Zepto (quick-commerce) product catalogue. The goal is to answer practical business questions about **pricing, discounts, stock availability and inventory value**.

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Dataset](#-dataset)
- [Tools Used](#-tools-used)
- [Repository Structure](#-repository-structure)
- [How to Run](#-how-to-run)
- [Workflow](#-workflow)
- [Business Questions Solved](#-business-questions-solved)
- [Key Findings](#-key-findings)
- [Limitations](#-limitations)
- [Future Improvements](#-future-improvements)

---

## 📖 Project Overview

This project covers the full SQL analysis cycle:

1. **Create** a table and load the raw CSV
2. **Explore** the data (row count, nulls, categories, stock status, duplicates)
3. **Clean** the data (remove invalid prices, convert paise to rupees)
4. **Analyse** the data with 8 business questions

**Skills shown:** `GROUP BY`, `HAVING`, `CASE WHEN`, aggregate functions, `DISTINCT`, `ROUND`, filtering, sorting, data cleaning with `UPDATE` / `DELETE`.

---

## 📂 Dataset 

- **File:** `zepto_v2.csv`
- **Size:** 3,732 rows × 9 columns
- **Categories:** 14 (e.g. Cooking Essentials, Munchies, Packaged Food, Personal Care, Fruits & Vegetables)
- **Stock status:** 3,279 in stock, 453 out of stock
- **Unique product names:** 1,681 (the same product can appear as multiple SKUs)
- **source:** https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset/data?select=zepto_v2.csv

| Column | Description |
|---|---|
| `sku_id` | Unique ID for each row (auto-generated in SQL, not in the CSV) |
| `Category` | Product category |
| `name` | Product name |
| `mrp` | Maximum Retail Price (in **paise** in the raw file) |
| `discountPercent` | Discount on MRP (%) |
| `availableQuantity` | Units available in stock |
| `discountedSellingPrice` | Price after discount (in **paise** in the raw file) |
| `weightInGms` | Product weight in grams |
| `outOfStock` | `TRUE` if the product is out of stock |
| `quantity` | Pack quantity / size value of the item |

---

## 🛠 Tools Used

- **Database:** PostgreSQL (uses `SERIAL`, `BOOLEAN`, `NUMERIC`)
- **Client:** pgAdmin / DBeaver / psql (any Postgres client works)

---

## 🗂 Repository Structure

```
├── zepto_v2.csv                 # Raw dataset
├── Zepto_SQL_data_analysis.sql  # Table creation, exploration, cleaning, analysis
└── README.md
```

---

## ▶️ How to Run

1. Create a PostgreSQL database.
2. Run the `CREATE TABLE` part of `Zepto_SQL_data_analysis.sql`.
3. Import the CSV into the `zepto` table. The CSV has **no** `sku_id` column, so list the columns when importing:

   ```sql
   COPY zepto (category, name, mrp, discountPercent, availableQuantity,
               discountedSellingPrice, weightInGms, outOfStock, quantity)
   FROM '/path/to/zepto_v2.csv'
   WITH (FORMAT csv, HEADER true, ENCODING 'UTF8');
   ```

   > If you use the import wizard in pgAdmin/DBeaver, skip `sku_id` and set the encoding to UTF-8.

4. Run the rest of the script (exploration → cleaning → analysis).

⚠️ **Run the cleaning step only once.** The `UPDATE` that divides prices by 100 will shrink prices again if you run it twice.

---

## 🔄 Workflow

### 1. Data Exploration
- Total row count and sample records
- Null value check across all columns
- List of distinct categories
- In-stock vs out-of-stock count
- Product names that appear more than once

### 2. Data Cleaning
- Found products with `mrp = 0` or `discountedSellingPrice = 0` and **deleted** the invalid rows
- **Converted paise to rupees** by dividing `mrp` and `discountedSellingPrice` by 100

### 3. Data Analysis
8 business questions answered with SQL (listed below).

---

## ❓ Business Questions Solved

| # | Question | Main SQL concepts |
|---|---|---|
| Q1 | Top 10 best-value products by discount % | `ORDER BY`, `LIMIT`, `DISTINCT` |
| Q2 | High-MRP products (> ₹300) that are out of stock | `WHERE`, boolean filter |
| Q3 | Estimated revenue per category | `SUM`, `GROUP BY` |
| Q4 | Products with MRP > ₹500 and discount < 10% | multi-condition `WHERE` |
| Q5 | Top 5 categories by average discount | `AVG`, `ROUND`, `LIMIT` |
| Q6 | Price per gram for products ≥ 100 g | calculated column, `ROUND` |
| Q7 | Group products into Low / Medium / Bulk by weight | `CASE WHEN` |
| Q8 | Total inventory weight per category | `SUM(weight × quantity)` |

### Sample query (Q5: average discount by category)

```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

---

## 📊 Key Findings

Numbers below are from the cleaned data (3,731 rows after removing one invalid row).

- **Discounts are low overall.** The average discount is about **7.6%**, and the median is 6%. The highest discount in the data is 51%.
- **Fresh categories discount the most.** *Fruits & Vegetables* leads with a **15.46%** average discount, followed by *Meats, Fish & Eggs* (**11.03%**). *Home & Cleaning* is the lowest at **5.70%**.
- **Best-value deals** are on Dukes Waffy wafers (51%), and several 50% deals on RRO cheese and Epigamia yogurt.
- **Most products are cheap.** The median MRP is about ₹110, and 3,392 of 3,731 rows (about 91%) weigh under 1 kg. Only 46 rows qualify as "Bulk" (5 kg or more).
- **About 12% of SKUs are out of stock** (453 of 3,732). Only **4 distinct products** with MRP above ₹300 are out of stock, including Patanjali Cow's Ghee (₹565) and Aashirvaad Atta with Multigrains (₹315).
- **Premium items get almost no discount.** 39 distinct products have MRP above ₹500 and a discount below 10%, such as Dhara Kachi Ghani Mustard Oil (₹1,250, 8%) and Saffola Gold (₹1,240, 0%).
- **Inventory value is concentrated.** *Cooking Essentials* and *Munchies* have the highest estimated stock value (about ₹3.37 lakh each), while *Fruits & Vegetables* has the lowest (about ₹10.8k).
- **Price per gram:** staples like salt and onion cost about ₹0.02/g, while hair oil and hair colour cost over ₹3.5/g.

---

## ⚠️ Limitations

Read these before using the numbers for any decision:

- **"Revenue" is not real revenue.** Q3 multiplies selling price by *available stock*. This is the **value of current inventory**, not sales. The dataset has no sales or order data.
- **Duplicate-looking categories.** Several categories have identical totals (for example *Cooking Essentials* and *Munchies*, or *Paan Corner* and *Personal Care*), and *Packaged Food*, *Ice Cream & Desserts* and *Chocolates & Candies* share the same average discount. This suggests the data may be partly synthetic or copied across categories, so category-level conclusions should be treated with caution.
- **Small stock numbers.** `availableQuantity` ranges only from 0 to 6, so this looks like a snapshot or sample, not full warehouse stock.
- **Price per gram loses detail.** Rounding to 2 decimals makes many cheap items show as `0.02`, so they cannot be ranked against each other.
- **Some weights are 0.** The data has `weightInGms = 0` rows. Q6 avoids division by zero only because it filters for 100 g or more.
- **Only one invalid row was removed**, and only the `mrp = 0` check is used in the `DELETE`. A row with `discountedSellingPrice = 0` but a valid MRP would not be removed.
- **No time dimension.** There are no dates, so trends cannot be analysed.

---

## 🚀 Future Improvements

- Rename Q3 to *"Estimated inventory value"* to be accurate
- Add `ORDER BY ... DESC` to Q3 and Q8 so the largest categories appear first
- Round price per gram to 3–4 decimals, or use price per 100 g
- Add a check for `weightInGms = 0` and handle the rows explicitly
- Compare MRP vs discounted price to confirm the `discountPercent` column is correct
- Add a Python or Power BI / Tableau dashboard on top of the SQL results
- Add window functions (`RANK()`) to find the top product within each category

---

## 👤 Author

**yagnesh**
[LinkedIn](https://www.linkedin.com/in/yagneshk88/) · [GitHub](https://github.com/yagneshhhh)

⭐ If you found this project useful, consider giving it a star.
