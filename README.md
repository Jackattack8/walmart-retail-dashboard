# Walmart Retail Analytics Dashboard

A retail vendor analytics dashboard built in Power BI on synthetic POS data modeled after a real Walmart Scintilla / Luminate Data Portal extract. Built to demonstrate practical supply chain and retail analytics skills in a vendor management context, including in-stock monitoring, weeks of supply, lost sales estimation, and phantom inventory detection.

---

## What This Is

Walmart vendors rely on their POS data. Every week a vendor analyst pulls a Scintilla extract and answers four questions for their buyer: 

- Are my items selling?
- Are they on the shelf when a customer wants them?
- Where is performance strong or weak across the store base?
- Do I have the right inventory in the right place, and is the data even telling the truth?

This dashboard answers all four across four pages built on a clean star schema data model.

---

## Dashboard Pages

### Page 1: Executive Summary
The 30 second view. Four KPI cards (Total POS Units, Total POS Dollars, Average In-Stock %, and Lost Sales Dollars shown in red as the headline business impact number), a weekly POS trend line showing seasonal patterns, Average In-Stock % by item, and a performance breakdown by district.

### Page 2: Sales Detail
The weekly business review view. Week over week POS trend with a WoW percent change line, POS dollars by subcategory, and interactive slicers for item and district filtering.

### Page 3: Instock and Inventory
The operational action view. A store level in-stock table with conditional formatting (red below 90 percent, yellow 90 to 95, green above 95), Pipeline Weeks of Supply by item with an overstock and out-of-stock risk target band, and Average In-Stock % by district.

### Page 4: Inventory Health
The data quality and business impact view. Cards for Lost Sales Units, Phantom Combos, and Average Sell Through %, a Phantom Inventory Risk table flagging store and item combinations that show healthy recent in-stock but near-zero recent sales, and a Phantom Risk Count by District chart. The distinction this page makes is deliberate: Page 3 shows a shelf presence problem where product is genuinely not there, while Page 4 shows a data integrity problem where the system says product is there but nothing sells.

---

## Data Model

Star schema with one fact table and three dimension tables, each dimension joined one to many onto the fact:

- Dim_Store joins to Fact_POS on Store Number
- Dim_Item joins to Fact_POS on Item Number
- Dim_Calendar joins to Fact_POS on Week Ending Date

**Fact_POS** holds one row per store, item, and week with POS Units, POS Dollars, On Hand Units, In Transit Units, In Warehouse Units, In-Stock %, and Sell Through %.

**Dim_Store** covers 90 stores across one Walmart region and 8 states, segmented by Volume Tier (High, Medium, Low) and District.

**Dim_Item** covers 40 SKUs across 5 subcategories, each carrying a replenishment health score that drives realistic in-stock variance.

**Dim_Calendar** covers a trailing 53 Walmart weeks. Walmart weeks run Saturday to Friday, so every week ending date is a Friday. The series ends on the most recently completed week and includes a Fiscal Year column following Walmart's February 1 to January 31 fiscal boundary.

---

## Key DAX Measures

| Measure | Description |
|---|---|
| Total POS Units | Sum of weekly units sold |
| Total POS Dollars | Sum of weekly revenue |
| Avg In-Stock Pct | Average in-stock percent across store and item combinations |
| Pipeline Weeks of Supply | On hand plus in transit plus in warehouse, divided by average weekly sales |
| WoW POS Units Pct | Week over week percent change in POS units |
| Baseline Weekly Velocity | Average units per store per item per week, used as the lost sales baseline |
| Lost Sales Units | Estimated units lost when weekly sales fall below the baseline velocity |
| Lost Sales Dollars | Lost Sales Units valued at average retail price |
| Phantom Combo Count | Count of store and item pairs with high recent in-stock but near-zero recent sales |

---

## Analytical Highlights

**Phantom inventory detection.** A standard weekly POS report would show these items as slow movers and move on. This dashboard cross references recent in-stock against recent sales to surface a different diagnosis: stock the system believes is on the shelf but that is not actually selling, the classic signature of shrinkage, miscounts, or mis-slotted product. Slow mover versus phantom inventory is a different root cause with a different fix.

**Lost sales estimation.** Converts an operational shortfall into a dollar figure a buyer cares about, by comparing actual weekly velocity against a baseline and valuing the gap at retail.

**Pipeline weeks of supply.** Goes beyond store on hand to include in transit and warehouse inventory, giving a full pipeline view rather than a store only snapshot.

---

## Data Generator

The synthetic dataset is produced by `generate_walmart_pos.py`, which builds a flat file extract mimicking what you would pull from Scintilla.

Realistic behaviors baked in:

- Seasonal index that lifts Q4 (holiday) and softens January and February
- Volume Tier multipliers that drive store level velocity differences
- 6 problem SKUs and 8 problem stores that create genuine in-stock variance
- Combined store and item health scores that drive replenishment frequency and out-of-stock probability so all three inventory metrics move together
- Seeded phantom inventory scenarios in the trailing weeks so the detection logic has real cases to flag

To generate your own dataset:

```bash
pip install pandas numpy faker
python generate_walmart_pos.py
```

Output is `walmart_pos_data.csv` at roughly 190,800 rows (90 stores by 40 items by 53 weeks).

---

## Tech Stack

- Power BI Desktop for the data model, DAX measures, and dashboard
- Power Query for star schema construction from the flat file source
- Python (pandas, numpy, faker) for synthetic data generation
- Data source modeled after a Walmart Scintilla / Luminate extract

---

## About

Built by Mark Burns, a supply chain and logistics professional with 20 plus years of experience in Walmart vendor operations, transportation management, and analytics. This project is part of a broader portfolio demonstrating practical applications of data tools in supply chain contexts.

[LinkedIn](https://www.linkedin.com/in/mark-k-burns/) | [Portfolio](https://supplychaintool-ten.vercel.app)
