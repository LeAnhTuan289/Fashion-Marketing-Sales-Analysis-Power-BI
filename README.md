# Fashion Marketing & Sales Analysis | Power-BI

![image](https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/d2fb426879c67d1bc1a9a5342ee63c87b19406b2/documents/Fashin_mar.jpg)

Author: Lê Anh Tuấn  

Tool Used: Power BI  

***

## 📑**Table of Contents**
1. [📌 Background & Overview](#background--overview)
2. [📂 Dataset Description & Data Structure](#dataset-description--data-structure)
3. [🧠 Design Thinking Process](#design-thinking-process)
4. [📊 Key Insights & Visualizations](#key-insights--visualizations)

## 📌 Overview

### 🎯 Project Objectives

This project aims to help senior managers monitor marketing budget allocation, evaluate campaign effectiveness, and identify which campaigns, product categories, and SKUs generate the strongest revenue and profitability impact.

The dashboard supports data-driven decisions on where to scale, optimize, reduce, or reallocate marketing budget to improve overall marketing efficiency.

### ❓ Business Questions

1. How is the marketing budget allocated and utilized across time, campaigns, product categories, and SKUs?

2. How effectively are marketing campaigns performing in terms of spend efficiency, revenue contribution, ROAS, CPC, and conversion performance?

3. Which campaigns, product categories, and SKUs are driving profitable growth, and where should the company scale, optimize, or reduce marketing investment?

👤**Target Audience**
- Marketing Team: to track marketing performance and decide which campaigns to scale, pause, or optimize.
- Head of Sales & Marketing: to view revenue, profit, and efficiency of marketing investment at a high level, compare channels (ads vs direct sales) and guide budget allocation

## 📂 Dataset Description & Data Structure
### 📌 Data Source

- Context: This dataset comes from a fashion company in Vietnam that operates mainly in retail and e-commerce. The company runs many multi-channel marketing campaigns (online and offline) to drive sales and build its brand.

1️⃣ **Tables**
- **Note: Dataset and column names are in Vietnamese, so this project focuses on documenting the table role and key fields instead of translating every column one by one.**
- Dataset includes 4 tables:
  - fact-order (3451 records): sale transations\
    Contains one row per order line, including:
    - Order info: ID, date, order status
    - Sales metrics: unit price, units sold, unit cost
    - Customer info: customer name, tier, birthday, city/province
    - Product info: product name, parent product code / barcode, product category
      
  - fact-mkt_camp_by_sku_cost (3844 records): Campaign × SKU performance\
    One row per campaign–product–day, including:
    - Date, product code / name
    - Campaign spend and allocated budget at SKU level
    - Media metrics: impressions, clicks, CPC, CPM, CTR
    - Engagement: inbox + comments (by ad manager/AM)
    - Performance: units sold via campaign, remaining inventory, etc.

  - dim-mkt_camp_cost (854 records): Campaign-level performance (one campaign can last many days and spend budget differently) \
    One row per campaign–day, including:
    - Campaign name, date, status
    - Daily budget and actual spend
    - Media KPIs: impressions, clicks, CPC, CPM, CTR
  - dim-danh sach san pham (dim-product_list) (2250 records): Product catalog\
    One row per product, including:
    - Product ID and barcodes
    - Product category and type
    - Product name
    - Cost fields: original cost, selling price, selling price with VAT
    - Attributes: status, colour, material, pattern, etc.
- Format: xlsx

2️⃣**Data Wrangling & Data Model Preparation**

**- Basic cleaning:**
  - Checked and corrected data types for all columns.
  - Removed duplicate rows and blank rows in all tables.
  - Dropped columns that were not needed for analysis.

**- Date dimension:**
  - Built a _dim_date_ table from _fact-order_:
    - Generated a continuous date range based on the min/ max order dates.
    - Added calendar attributes: Date, Year, Quarter, Month, Month Name, Week, Week number, Weekday, Day, Day Name, etc.
    - Linked dim_date to both fact-order and fact-mkt_camp_by_sku_cost tables so time-based analysis (by day, week, month) is consistent across the model.
      
**- Campaign keys (Primary & Foreign Key):**
  - In _dim-mkt_camp_cost_:
    - Created a custom column by combining **Campaign Name + Date + Budget Spent**.
    - Added an index on this column and used it as a unique **Campaign Id** (primary key).
  - In _fact-mkt_camp_by_sku_cost_:
    - Merged _fact-mkt_camp_by_sku_cost_ with _dim-mkt_camp_cost_ on **Campaign Name + Date + Budget Spent**.
    - Expanded the merge to bring **Campaign Id** into _fact-mkt_camp_by_sku_cost_.
    - **Campaign Id** now acts as a foreign key linking campaign costs and SKU-level performance.
   
 **- Tagging orders as Ads Sales vs Direct Sales:**
  - Table: fact-order
      - This table contains all orders (both store / organic sales and campaign-driven sales).
      - Goal: identify which orders came from campaigns to separate Ads Revenue from Direct Revenue.\
  - Steps:
     - Create a helper table
        - Duplicated_ fact-mkt_camp_by_sku_cost_ → _fact-mkt_camp_by_sku_cost (2)_.
        - Kept only **Date** and **Product Code**, then removed duplicates.
        - Result: a list of **Product Code + Date** combinations that were sold via campaigns.
     - Merge with orders
        - Merged_ fact-order_ with _fact-mkt_camp_by_sku_cost (2_) on **Date + Product Code**.
        - Expanded the merge to bring back **Product Code** from the campaign table.
      - Classify channel
        - Added a conditional column **Ads/Direct** in _fact-order_:
          - If the merged **Product Code** is not null → "Ads Sales".
          - If it is null → "Direct Sales".


3️⃣**Data Relationship**

From the data tables above, I created Star Schema Model as below

<img width="830" height="740" alt="Data model" src="https://github.com/user-attachments/assets/190b0cfa-d2aa-4516-82db-44ace683c55c" />

**Relationships between tables**

| From Table | To Table | Join Key | Relationship Type |
|----------|----------|----------|----------|
|dim_date| fact-order| dim_date[Date] → fact-order[Thời gian]| One to many (1:*)|
|dim_date| fact-mkt_camp_by_sku_cost| dim_date[Date] → fact-mkt_camp_by_sku_cost[Ngày]| One to many (1:*)|
|dim-product_list| fact-order| dim-product_list[Mã sản phẩm] → fact-order[Mã sản phẩm cha]| One to many (1:*)|
|dim-product_list| fact-mkt_camp_by_sku_cost| dim-product_list[Mã sản phẩm] → fact-mkt_camp_by_sku_cost[Mã Sản phẩm]| One to many (1:*)|
|dim-mkt_camp_cost| fact-mkt_camp_by_sku_cost|dim-mkt_camp_cost[Campaign Id] → fact-mkt_camp_by_sku_cost[Campaign Id]| One to many (1:*)|

## 🧠 Design Thinking Process

### 1️⃣ Empathize

![Image](https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/e7a0a551a71b8473942029dbf9c89e333c36ed18/documents/5W1H.png)

![Image](https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/e7a0a551a71b8473942029dbf9c89e333c36ed18/documents/EMPATHY.png)

### 2️⃣ Define point of view 

![Image](https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/e7a0a551a71b8473942029dbf9c89e333c36ed18/documents/POV.png)


## 📊 Key Insights & Visualizations

### 🔍 Dashboard Preview

1️⃣ **Overview**

<img width="1641" height="924" alt="Overview" src="https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/d2fb426879c67d1bc1a9a5342ee63c87b19406b2/documents/db1.png" />

- **Sales by Channel (Ad vs Direct Sales):**
  - Total revenue is ~5bn, of which ~3bn comes from ad-driven sales => roughly 60% of revenue is ad-driven, and 40% of revenue is from direct sales. However, Gross Margin ROAS (GM ROAS) is only 0.9, meaning every 1 VND of ad spend brings back less than 1 VND in gross margin. Campaigns are therefore critical for scale but not yet efficient.
    
- **How is Campaign Budget allocated?**
  - Spend is highest in Weeks 2–>4, with overall budget utilization around 80–90% and a few days of clear overspend.But overspending days do not consistently line up with higher profit or GM ROAS, and the scatterplot shows no strong link between high total spend and high Gross Margin ROAS. This suggests room to reallocate budget away from low-GM ROAS campaigns rather than just spending more.
    
- **Do campaigns and SKUs drive profitable growth?**
  - Across most product categories, ads drive more revenue than direct sales, confirming that campaigns successfully push product. Yet many high-revenue categories show low or even negative GM ROAS (e.g., Áo Tách Set/ Separate Top, Váy Chiết Eo Xòe/ Fit-and-Flare Dress) while some smaller-revenue categories are more profitable (e.g., Váy Chiết Eo Ôm/ Bodycon Dress). In other words, we are buying revenue in some areas but not necessarily profitable growth, likely due to COGS and discount structure on those products.

2️⃣**Campaign Performance**

<img width="1181" height="663" alt="Campaign Performance" src="https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/d2fb426879c67d1bc1a9a5342ee63c87b19406b2/documents/db2.png" />

- **Overview:**
  - Overall, Ads spend, budget utilization and ads revenue peak in weeks 2->4.
  - Over the same period, Cost Per Acquisistion (CPA) increases while orders through campaigns decline, indicating that each conversion is getting more expensive even as we spend heavily.

- **Which campaigns are doing good/ bad?**
  - Most campaigns cluster between -2.5 and 5 GM ROAS.
  - A few campaigns show high spend and high revenue but negative GM ROAS (e.g. AUDREY SHIRT). These are effectively buying revenue at a loss.
  - On the other hand, some campaigns have lower spend and smaller revenue but much higher GM ROAS (e.g. KINO DRESS, LISA DRESS, MARGNET DRESS), meaning they generate more margin per VND spent.
  - The _Top/ Bottom 10 Campaigns with Highest/ Lowest GM ROAS_ chart and _Campaign Performance_ table highlights:
      - Top campaigns and products that deliver strong GM ROAS and are candidates for scaling.
      - Bottom campaigns with weak GM ROAS and/or very high Ads spend that should be fixed or downsized.
  - This supports decisions to reallocate budget away from high-spend, low-ROAS campaigns and towards high-ROAS, under-funded campaigns and SKUs.
    
- **Campaign Funnel & Engagement**
  - Volume:
      - Impressions: ~5.1M
      - Clicks: ~41.6K → CTR ≈ 0.8%
      - Inbox + Comments: ~11.4K
      - Orders: ~1,684 → overall conv ≈ 0.03% of impressions
  - Cost per step (VND):
      - CPM ≈ 77K, CPC ≈ 9.5K, Cost Inbox+Comment ≈ 34K, CPA ≈ 234K 
  => CPM and CPC are moderate, but very few impressions ultimately become orders, and CPA is high. The main problem sits with click/ inbox and final purchase, not at the media buying impressions. 
  => Recommend to optimize landing pages, checkout flow, test offers, product mix, retarget on engaged users (clickers, inbox/comment) to yield more conversion and turn more users into buyers. 

3️⃣**Product:**

<img width="1184" height="664" alt="Customer and product" src="https://github.com/LeAnhTuan289/Fashion-Marketing-Sales-Analysis-Power-BI/blob/d2fb426879c67d1bc1a9a5342ee63c87b19406b2/documents/db3.png" />

- **By product: where do ads really matter?**
  - For most categories, ads drive more revenue than direct sale, especially Áo Tách Set/ Separate Top, Váy Chiết Eo Xòe/ Fit-and-Flare Dress, Set Quần Áo/ Outfit Set Top &  Bottom have the highest units sold and are heavily driven by ads => These are the first places to optimise campaign structure, targeting and creatives.
  - Some categories, for example, Áo Khoác/ Jacket, Quần Tách Set/ Separate Pants have low volume overall and limited contribution from ads.
  - Áo khoác (Jacket) has all orders (2 units ordered) from direct sales, none from ads.

- **Are we allocating budget to the right products?**
  - The dress categories, like Váy Chiết Eo Ôm/ Bodycon Dress, Váy Suông Xòe/ Loose Flared Dress, Váy Chiết Eo Xòe/ Fit-and-Flare Dress, Set váy áo/ Skirt & Top Set, sit in good area with solid budget utilization, above-average GM ROAS and healthy unit volume. These categories should be gradually scaled in campaigns.
  - Áo Tách Set/ Separate Top stands out with high budget used and large volume but negative GM ROAS. This means we push a lot of budget into a category that doesn’t pay back in margin.
  - Key take-away: Budget is concentrated in the right product family (dresses). However, we have over-funded low-margin categories (Áo Tách Set/ Separate Top) and under-funded high-margin ones that could be scaled.
      - Each product has its own drill-through page showing unit economics, channel performance over time, customer tier & age, best campaigns (high GM ROAS) and preferred variants (e.g., colors, materials). This effectively turns every product into a decision card: scale, fix, or cut.

### Recommendations

1️⃣**Reallocate budget to what actually earns margin:**

- Reduce budget on campaigns and products with high spend but negative GM ROAS, especially Áo Tách Set/ Separate Top and low-margin campaigns like AUDREY SHIRT.
- Reallocate that budget to dress categories with positive GM ROAS and reasonable CPA (Váy Chiết Eo Ôm/ Bodycon, Váy Chiết Eo Xòe/ Fit-and-Flare, Váy Suông Xòe/ Loose Flared, Set váy áo/ Skirt & Top Sets), so more money goes into products that actually generate profit, not just revenue.

2️⃣**Make the funnel more efficient from impression to order:**

- Review campaigns with low CTR and high spend and either pause them or refresh audiences and creatives so we pay for fewer “empty” impressions.
- For top-spend campaigns, check the landing and product pages (e.g., pricing, stock, messaging) and run simple A/B tests to improve conversion after the click.
- Create basic retargeting campaigns for people who clicked or messaged but did not buy, using creatives that push best-margin products and current offers.

3️⃣**Use each product’s drill-through page as a decision card:**
- For every main product category, look at its GM ROAS, CPA, inventory %, and who buys it on the drill-through page before making budget decisions.
- Scale products that have good GM ROAS and enough stock, fix or re-position products with high volume but low margin, and quietly stop promoting products that are weak on both sales and margin.

4️⃣**Bring customer segments into the decisions:**
- Use the _Product's drill through_ page to see which customer tiers and age groups respond best to each product family, and tailor campaigns accordingly (for example, dresses towards the strongest tier or age band).
- For segments that mainly convert via direct sales, rely more on organic, CRM and store activities, and for segments that respond well to ads, focus budgets on the products that show profitable GM ROAS for them.
