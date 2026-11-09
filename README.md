# Walmart Hurricane Project

## Team Members
* Amilyn Estes
* Vi Nguyen
* Adeline Kwakye

---

## 1. Project Overview & Objective
This project analyzes sales data from Walmart stores in Florida before and during Hurricane Helene. The goal is to clean and organize the transaction data, standardize product and store information, and identify which products and categories experienced the largest increases in demand. These findings would show how hurricanes affect shopping patterns and help stores better prepare for future disasters.

---

## 2. Analysis Windows 
* **Landfall:** Hurricane Helene landfall on **September 26, 2024**
* **Baseline Window:** **September 1, 2024 – September 15, 2024 (15 days)**
  * *Reasoning:* This period represents normal shopping patterns over two full weeks, prior to hurricane warnings and preparations affecting customer purchases.
* **Surge Window:** **September 19, 2024 – September 25, 2024 (7 days)**
  * *Reasoning:* Covers the seven days leading up to the hurricane, when customers started buying more supplies to prepare for the storm.

---

## 3. Key Findings
* **Top 5 Surging Merchandise Categories**: breakfast, emergency, entertainment, alcoholic beverages, and health. Breakfast had the highest relative surge with a 5.15 surge multiple ( +415.45%), while emergency had the highest absolute volume surge (+186.8%).
* A rise in sales was seen starting on September 19th, with most stores experiencing purchasing spikes by September 21st, and peaked sharply about 24–48 hours prior to landfall (September 24–25). Demand, however, plummeted immediately on September 26th, potentially reflecting store closures, power loss, and sheltering during the hurricane.
* The strongest correlations occurred at a 2-day lag, suggesting Walmart sales increased approximately two days before stronger winds and lower barometric pressure.
* All 12 stores experienced substantial increases in sales, ranging from approximately 125.05% to 173.00%. The largest surge occurred at FL-330 in Miami, with a 173.00% increase, followed by FL-619 in Tallahassee (160.05%) and FL-105 in Tampa (154.53%). The smallest increase occurred at FL-239 in Orlando (125.05%).
* Even when removing lumped electronics, actual Board Games and Cards (Monopoly and playing cards) alone experienced a massive ~294% surge in daily sales, leaping from a baseline of ~ 1,100/ day to a peak of 4,337.52 on September 25th.
* Emergency electronics (like portable chargers) and board games spike in tandem during the same pre-landfall window. 

---

## 4. Data Cleaning & Pipeline Architecture

All cleaning decisions were tracked through a decision log. The dataset was sanitized through the following dedicated modules:

1. **Exact Duplicate Removal (`handle_duplicates`):**
   * Identified and dropped 30 exact duplicate rows, reducing transactions from 25,562 to 25,532.
2. **Currency Standardization (`clean_unit_price`):**
   * Stripped currency symbols (`$`) and whitespace, converting price strings to numeric floats (`float64`).
3. **Negative Quantity Policy (`handle_negative_values`):**
   * Retained 130 negative quantity transactions to account for any returns/refunds. 
4. **Store Identifier Normalization (`clean_store_id`):**
   * Standardized irregular store keys across both transaction and store location tables to the canonical format `FL-###` (verified 100% store ID match).
5. **Timestamp Parsing & Imputation (`clean_timestamps`):**
   * Resolved mixed timestamp formats and ISO `T` separators, normalized date values, and dropped 884 records with unresolvable null timestamps, resulting in 24,648 clean transaction rows.
6. **SKU & Product Name Consolidation (`mapping_rules` & `standardize_product_names`):**
   * Resolved 45 raw SKU codes and 63 fragmented product name strings down to 20 canonical SKUs and unique product titles using substring pattern matching and mode-based imputation.
7. **Category Canonicalization (`category_map` & `category_remap_extra`):**
   * Consolidated 37 raw casing and spacing variants into standard merchandise categories.


---

## 5. Weather API
To evaluate spatial vulnerability, store latitude and longitude coordinates were matched against historical weather observations using the **Open-Meteo Historical Weather API** (`https://archive-api.open-meteo.com/v1/archive`). Daily maximum wind speed and total precipitation were retrieved across Florida store locations for September 1–30, 2024.


