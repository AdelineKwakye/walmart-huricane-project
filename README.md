# Walmart Hurricane Project

## Team Members
* Amilyn Estes
* Vi Nguyen
* Adeline Kwakye

---
## How To Run
* **Runs in Google Colab**
  * In repository navigate to `notebooks/team_notebook.ipynb`
  * Click the **"Open in Colab"** badge at the top of `team_notebook.ipynb`
  * In Colab, select **Runtime -> Run all** from the top menu to execute all cells sequentially from top to bottom.
    * *Note on Data & APIs:* No manual file uploads or API keys are required. The notebook automatically loads all four datasets directly from this repository's `data/` directory and fetches historical weather data from the public Open-Meteo Archive API (internet access required).

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

---
## 6. Reflection 

### 1. Data Quality Impact
* **Issue & Resolution:** A specific data quality issue that we encountered was inconsistent category names with unstandardized formatting (for example, `'breakfastfoods'` vs. `'breakfast'`). Despite these representing the exact same product group, spelling differences caused them to be counted as separate categories. To address this, we implemented explicit category remapping and text normalization dictionaries. Consolidating these fragmented transactions revealed that `breakfast` had the highest relative surge (**+415.4%**), while `emergency` (**+$5,720.42/day**) and `entertainment` (**+$4,455.05/day**) dominated total daily sales.
* **Impact of Leaving Data Uncleaned:** If we had grouped the data without category consolidation, our Top 5 surging categories would have been inaccurate. `Breakfast`—which appeared in four different ways—would have occupied four of the Top 5 spots instead of appearing once as a single consolidated category. This fragmentation distorts true category rankings, lowers daily averages, and masks the true **5.15x** surge multiple.

### 2. ETL Tradeoffs
* **Current Decision:** In our pipeline, we chose to drop all transactions that had null or unparseable timestamps (affecting **884 rows** out of 25,532).
* **Alternative Approach & Risk:** Instead of discarding these 884 transactions, we could have assigned missing records to the overall store-level median date or categorized them under a special `"unknown date"` bin for aggregated analysis. By discarding 884 transactions entirely, we underestimated overall sales volume by approximately **3.5%**. If those missing timestamps occurred disproportionately during the high-velocity pre-landfall period (September 19–25), our calculation of peak demand may have been deflated. Imputing with a store-level median would retain overall sales volume and allow us to adjust the surge window accordingly.

### 3. Pipeline Trust
* **Garbage In, Garbage Out:** We agree with the Walmart CDO—the algorithm will process whatever input it receives without warning you whether the input is accurate. Throughout this project, our core findings (such as the **5.15x** surge multiple) depended entirely on cleaning the data before building any model or summary table. Without thorough standardization, downstream outputs are neither accurate nor representative of true retail behavior.
* **Most Fragile Pipeline Component:** The most fragile part of our pipeline was inconsistent SKU and product-name string fragmentation. A major portion of our cleaning process involved identifying naming variations (such as multiple versions of *Strawberry Pop-Tarts*) and standardizing them while preserving physical unit quantities. If the consolidation step accidentally dropped or duplicated rows during remapping, it would distort unit counts and misrepresent inventory—leading to stockouts of critical goods or over-allocation of low-demand items during a crisis.

### 4. Business vs. Data
* **Statistical Finding vs. Inventory Risk:** After separating out lumped electronics (Roku sticks and portable chargers), our data still showed an impressive **~294% surge** in analog board games and playing cards. However, board games are traditionally low-velocity, high-cube items; over-stocking them ahead of a storm risks tying up shelf space and capital in inventory that may sit unsold for months if the storm shifts or power isn't lost.
* **Mitigation Strategy:** To balance statistical evidence with business risk, we proposed a **cross-merchandising "Power Outage Preparedness" endcap**, positioning slow-moving analog games directly next to high-velocity emergency electronics (such as portable chargers). This places games alongside high-foot-traffic essentials, improving clearance odds without wasting dedicated aisle space.
* **Value of Additional Data:** With store-specific inventory and stockout rate data, we could localize this recommendation—target-stocking only stores in high-probability power outage zones or regions with larger family demographics rather than applying a blanket increase across all 12 Florida stores.

### 5. Team Collaboration
* **Workflow & Future Iteration:** Our team divided the workload by assigning each member specific questions to research and analyze, allowing everyone to contribute equally and work efficiently. We also peer-reviewed one another's code and outputs to ensure accuracy, consistency, and quality throughout the project. Given an additional week, we would spend more time refining our statistical analysis, polishing visualizations, and exploring secondary trends to further strengthen our findings.
* **Key Team Contributions:**
  * **Vi:** Strong communication and adaptability across tasks.
  * **Amilyn:** Strong dependability and collaborative execution.
  * **Addie:** Proactive work ethic and deliverable ownership.


