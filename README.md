# 📊 Nexus Retail Group Ltd. — E-Commerce Omnichannel Growth & Unit Economics Intelligence

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Microsoft Excel](https://img.shields.io/badge/Microsoft_Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![Data Modeling](https://img.shields.io/badge/Data_Modeling-Star_Schema-blue?style=for-the-badge)
![DAX](https://img.shields.io/badge/Analytics-Advanced_DAX-orange?style=for-the-badge)

---

## 📌 Executive Summary

**Nexus Retail Group Ltd.** is an omnichannel retail enterprise operating across multiple commercial categories (Electronics, Home & Office, Lifestyle, Fitness & Apparel) and global distribution hubs (North America, Europe, Asia-Pacific).

This analytics project delivers an end-to-end data transformation pipeline auditing **32,000 multi-market customer orders** and comprehensive paid marketing streams ($1.57M ad spend). The workflow executes foundational data cleansing and financial reconciliation in Microsoft Excel, defines an enterprise Star Schema model, and surfaces insights across a 4-page interactive dashboard suite built in **Power BI Desktop**.

---

## 📁 Repository Directory Structure

```text
e commerce dataset/
│
├── 📂 Clean Table/                     # Audited, normalized tables formatted for Power BI
│   ├── Fact_Orders.xlsx                # Orders master table with cohort indices & revenue totals
│   ├── Fact_Order_Items.csv            # Line-item SKU pricing, quantities, discounts, and margins
│   ├── Fact_Marketing_Spend.csv        # Daily ad costs, impressions, clicks, CTR, and CPC metrics
│   ├── Dim_Products.csv                # SKU catalog, unit costs, pricing, and category hierarchies
│   ├── Dim_Customer.csv                # Customer profiles, segment classification, and loyalty tiers
│   ├── Dim_MarketingCampaigns.csv      # Campaign metadata, platform mapping, and target CPAs
│   └── Dim_Regions.csv                 # Geographic regions, tax percentages, and warehouse links
│
├── 📂 Raw Dataset/                     # Unprocessed source files prior to ETL and audits
│   ├── Raw_Orders.csv                  # Raw order logs containing tax and shipping discrepancies
│   ├── Raw_Order_Items.csv             # Unfiltered line items containing orphan/unmapped SKUs
│   ├── Raw_Marketing_Spend.csv         # Raw ad spend logs with unformatted date serials
│   ├── Raw_Customers.csv               # Raw customer registrations
│   ├── Raw_Products.csv                # Raw product catalog
│   ├── Raw_MarketingCampaigns.csv      # Unmapped campaign IDs and split channel source names
│   └── Raw_Regions.csv                 # Unstandardized regional tax rates
│
├── 📂 powerbi/                         # Core Power BI solution
│   └── Nexus_Retail.pbix               # Fully configured 4-page Power BI dashboard file
│
└── 📂 screenshot/                      # High-resolution dashboard captures
    ├── page1.png                       # Page 1: Executive Omnichannel Overview
    ├── page2.png                       # Page 2: Paid Attribution & Unit Economics
    ├── page3.png                       # Page 3: Customer Cohort Retention & Repeat Curve
    └── page4.png                       # Page 4: Product Profitability & Basket Intelligence
```

---

## 🛠️ Data Pipeline & Cleaning Methodology

1. **Regional Tax Rate Recalibration**:
   * *Issue*: Decimal percentage inputs in regional records suffered from repeated division by 100, resulting in nominal values like `0.07%` ($0.0007$).
   * *Resolution*: Standardized decimal rates in `Dim_Regions` (e.g., 7.0%, 20.0%) and re-audited order totals:
     $$\text{Order Grand Total} = \text{Net Revenue} + \text{Shipping Fee} + (\text{Net Revenue} \times \text{Tax Rate})$$
2. **Dynamic Conditional Shipping Fee Logic**:
   * *Issue*: String text hyphens (`"-"`) present in zero-cost shipping rows triggered `#VALUE!` errors during arithmetic summation.
   * *Resolution*: Formatted numeric binary shipping thresholds: orders $\ge \$75$ received free shipping ($0.00), while orders below $\$75$ evaluated to a flat $\$6.99$ charge.
3. **Orphan SKU Isolation & Unit Rectification**:
   * *Issue*: Unmapped catalog records (e.g., `SKU-UNKNOWN-99`) distorted profit margin ratios.
   * *Resolution*: Isolated unmapped item lines to zero-cost/zero-revenue status, locking validated unit sales to **89,620 units** with no visual gaps.
4. **Marketing Attribution Channel Unification**:
   * Resolved fragmented channel strings (e.g., `Meta` vs. `Meta Ads`, `Google Ads` vs. `Google Search`) into 5 consolidated attribution streams: **Google Ads**, **Meta Ads**, **Affiliates**, **Email Newsletter**, and **Organic / Direct**.

---

## 📐 Data Modeling (Star Schema Architecture)

Data relationships strictly maintain a **Star Schema** with one-way single-directional filters ($1 \rightarrow *$) to prevent cross-filter ambiguities:

```text
       [Dim_Date]               [Dim_Customer]             [Dim_Regions]
           │ 1                       │ 1                        │ 1
           │                         │                          │
           └───* [Fact_Orders] <─────┘                          │
                     │ 1                                        │
                     │                                          │
                     ├───* [Fact_Order_Items] >────* [Dim_Products]
                     │
           ┌─────────┴─────────┐
           │ *                 │ *
  [Fact_Marketing_Spend]   [Dim_MarketingCampaigns]
```

* **`Dim_Date`**: Custom DAX calendar table generated across the active business horizon (`2024-01-01` to `2025-12-31`), containing year, quarter, month-year text, and integer sorting columns.

---

## 📈 Key Metrics & DAX Measures

### Financial & Volume Measures
```dax
Total Net Revenue = SUM(Fact_Orders[Order_Net_Revenue])
```
```dax
Total Orders = DISTINCTCOUNT(Fact_Orders[Order_ID])
```
```dax
AOV = DIVIDE([Total Net Revenue], [Total Orders], 0)
```
```dax
Total Gross Profit = SUM(Fact_Order_Items[Line_Gross_Profit])
```
```dax
Gross Margin % = DIVIDE([Total Gross Profit], [Total Net Revenue], 0)
```

### Marketing & Unit Economics Measures
```dax
Total Ad Spend = SUM(Fact_Marketing_Spend[Clean_Daily_Cost])
```
```dax
Blended CAC = DIVIDE([Total Ad Spend], [Total Orders], 0)
```
```dax
ROAS = 
IF(
    [Total Ad Spend] = 0, 
    BLANK(), 
    DIVIDE([Total Net Revenue], [Total Ad Spend])
)
```

### Customer Retention & Lifecycle Measures
```dax
Total Customers = DISTINCTCOUNT(Fact_Orders[Clean_Customer_ID])
```
```dax
Repeat Customer Orders = 
CALCULATE([Total Orders], Fact_Orders[Cohort_Index] > 0)
```
```dax
Repeat Order Rate % = DIVIDE([Repeat Customer Orders], [Total Orders], 0)
```

---

## 🖥️️ Dashboard Architecture & Visual Analytics

### Page 1: Executive Omnichannel Overview
* **Focus**: Executive financial health, top-line performance, and geographic distribution.
* **Core KPIs**: Net Revenue ($5.93M), Total Orders (32K), AOV ($185.26), Gross Margin (57.97%), Blended ROAS (3.78x).
* **Visuals**:
  * *Monthly Net Revenue vs. Ad Spend Trend*: 24-month chronological progression linking monthly revenue spikes against marketing spend.
  * *Revenue Share by Payment Type*: Donut visual highlighting Credit Card (42.9%), Apple Pay (24.73%), Klarna BNPL (17.89%), PayPal (11.64%), and Other (2.84%).
  * *Net Revenue by Product Category*: Horizontal bar chart ranking Electronics ($2.8M), Home & Office ($1.8M), Lifestyle ($0.8M), and Fitness & Apparel ($0.5M).
  * *Regional Revenue Distribution*: Geographic breakdown tracking revenue delivery across Europe-DACH, North America-Central, Europe-UK, and other territories.

### Page 2: Paid Attribution & Unit Economics
* **Focus**: Acquisition efficiency, campaign performance, and channel-level ROAS.
* **Core KPIs**: Total Ad Spend ($1.57M), Blended CAC ($49.07), Overall ROAS (3.78x), CTR (2.82%), CPC ($0.57).
* **Visuals**:
  * *CAC vs. ROAS Efficiency Matrix*: Scatter quadrant mapping organic/low-cost drivers (Email Newsletter at 24.10x ROAS / $7.81 CAC; Affiliates at 8.52x ROAS / $22.27 CAC) against scaled paid traffic channels (Meta Ads at 2.54x ROAS / $72.74 CAC; Google Ads at 2.21x ROAS / $83.56 CAC).
  * *Ad Spend vs. Net Revenue Attribution*: Dual-bar comparison of direct channel investments against generated returns.
  * *Marketing Channel Performance Matrix*: Granular table summarizing spend, volume, top-line revenue, gross margins, CAC, and channel-specific ROAS.

### Page 3: Customer Cohort Retention & Repeat Curve
* **Focus**: Cohort lifecycle survival, retention decay rates, and order re-engagement.
* **Core KPIs**: Total Customers (4,500), Repeat Customer Orders (27K), Repeat Order Rate (83.69%), Average Orders per Customer (7.11).
* **Visuals**:
  * *Monthly Cohort Retention Heatmap*: Matrix tracking retention drop-off month-by-month across 24 distinct acquisition cohorts.
  * *Customer Retention Decay Curve*: Multi-cohort trend line tracking customer activity stabilization from Month 0 to recurring retention baselines.
  * *New vs. Repeat Orders by Month*: 100% stacked column visual validating a customer lifecycle shift from 100% new acquisitions in Month 1 to recurring order dominance over time.

### Page 4: Product Profitability & Basket Intelligence
* **Focus**: SKU margins, product mix contribution, and pricing sensitivity.
* **Core KPIs**: Total Gross Profit ($3.44M), Gross Margin % (57.97%), Average Basket Size (1.38 items), Total Discount Amount ($438.32K).
* **Visuals**:
  * *Gross Profit Contribution by SKU Sub-Category*: Treemap segmenting product margins across categories and sub-categories.
  * *Price vs. Sales Volume (Elasticity)*: Scatter plot evaluating unit volume sensitivity relative to catalog retail price points.
  * *SKU Level Performance Table*: Product profitability register displaying retail prices, unit costs, clean unit sales (89,620 total), line revenue, gross profit, and percentage margins supported by dynamic green status indicators.

---

## 🚀 How to Open and Review

1. Clone or download the repository to your local directory (`Desktop > e commerce dataset`).
2. Open `powerbi/Nexus_Retail.pbix` in **Power BI Desktop**.
3. If source paths need updating on your machine, select **Transform Data > Data source settings** and update the path to point to your local `Clean Table/` folder.
