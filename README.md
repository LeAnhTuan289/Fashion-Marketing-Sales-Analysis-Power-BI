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

---

## 1️⃣ Executive Overview

<img width="1641" height="924" alt="Executive Overview" src="documents/db1.png" />

### Business Questions Answered

- How is the marketing budget allocated and utilized over time?
- Is marketing spend translating into revenue and profit?
- How much does marketing contribute to total business revenue?
- Which time periods show stronger or weaker marketing efficiency?

### Key Insights

- **Marketing is the main revenue driver.**  
  Marketing generated **3.02bn revenue**, contributing **63.34% of total revenue**, while direct revenue reached **1.75bn**. This shows that marketing campaigns play a major role in driving sales for the business.

- **Overall ROAS is strong, but profitability is not stable.**  
  Overall Marketing ROAS reached **7.67**, meaning every 1 VND spent on marketing generated around 7.67 VND in marketing-attributed revenue. However, profit margin fluctuates significantly by week, showing that high revenue does not always translate into stable profit.

- **The most efficient week is not necessarily the highest-spend period.**  
  Weekly Marketing ROAS peaks at around **9.05**, while other weeks stay around **5.72–7.63**. This suggests that campaign mix and product mix matter more than simply increasing spend.

- **Some campaigns show high budget usage but do not always generate proportional revenue.**  
  The scatter plot between budget used and marketing revenue indicates that higher budget usage does not always guarantee stronger revenue performance.

### Recommendations

#### 1. Strengthen revenue channels beyond paid campaigns

- **Where to act:** Revenue mix between marketing-driven revenue and direct revenue.
- **What to do:** Maintain high-performing campaigns, but also build stronger direct and retention-based channels such as email marketing, loyalty programs, remarketing, customer reactivation, and organic content.
- **Why it matters:** More than 60% of revenue comes from marketing-driven sales, so the business becomes vulnerable if ad costs rise or campaign performance drops.
- **Goal:** Keep marketing as a growth driver while reducing overdependence on paid campaigns.

#### 2. Manage weekly budget based on both ROAS and profit margin

- **Where to act:** Weekly campaign budget allocation and profit margin performance.
- **What to do:** Review weeks or campaigns with high revenue but weak profit margin. Combine ROAS with profit margin, discount level, product cost, and marketing spend before deciding whether to scale.
- **Why it matters:** A campaign can generate high revenue but still be inefficient if marketing costs or product costs are too high.
- **Goal:** Improve marketing efficiency by ensuring that revenue growth also creates healthy profit.

#### 3. Replicate the most efficient weekly campaign pattern

- **Where to act:** Campaign and product mix from the highest-ROAS week.
- **What to do:** Identify which campaigns, SKUs, and categories contributed to the strongest ROAS week, then use them as a benchmark for future budget planning.
- **Why it matters:** The dashboard shows that efficiency comes from the right campaign/product mix, not just higher spending.
- **Goal:** Improve average ROAS by scaling proven high-efficiency patterns.

---

## 2️⃣ Campaign Performance

<img width="1181" height="663" alt="Campaign Performance" src="documents/db2.png" />

### Business Questions Answered

- Which campaigns generate the highest marketing revenue?
- Which campaigns perform best or worst based on ROAS, CPC, conversion performance, and revenue contribution?
- Where are the main bottlenecks in the marketing funnel?
- Is marketing revenue concentrated in only a few top campaigns or spread across many campaigns?

### Key Insights

- **Revenue is not concentrated only in the Top 10 campaigns.**  
  The Top 10 campaigns generated **1.04bn**, accounting for **34.28%** of marketing revenue, while the remaining campaigns contributed **1.99bn**, or **65.72%**. This means long-tail campaigns play a major role in total marketing revenue.

- **The funnel loses many users from impressions to clicks.**  
  The dashboard shows around **5.11M impressions** but only **41.85K clicks**, meaning the click-through stage is a key bottleneck. Many users see the ads, but only a small share decide to click or interact.

- **High-revenue campaigns are not always the most efficient campaigns.**  
  For example, **AUDREY SHIRT** generated the highest marketing revenue, but some smaller campaigns such as **LISA DRESS** and **MARGNET DRESS** delivered much stronger ROAS. This means campaign decisions should not be based on revenue alone.

- **The current conversion rate definition may confuse business users.**  
  The dashboard shows **Conversion Rate = 466.40%**. If this is not a standard click-to-order or visit-to-order conversion rate, the metric should be renamed or clearly explained to avoid misunderstanding.

### Recommendations

#### 1. Optimize the full campaign portfolio, not only the Top 10 campaigns

- **Where to act:** Campaign portfolio, especially campaigns outside the Top 10.
- **What to do:** Segment campaigns into four action groups: **Scale**, **Optimize**, **Reduce**, and **Stop/Test Again** based on ROAS, spend, revenue contribution, CPC, and conversion performance.
- **Why it matters:** Campaigns outside the Top 10 contribute more than 60% of marketing revenue, so focusing only on top campaigns would miss many optimization opportunities.
- **Goal:** Improve overall campaign portfolio efficiency and avoid wasting budget on weak long-tail campaigns.

#### 2. Improve the top-of-funnel conversion from impressions to clicks

- **Where to act:** Impression → Click stage.
- **What to do:** A/B test ad creatives, product visuals, headlines, offers, CTAs, and audience targeting. Prioritize campaigns with high impressions but low click performance.
- **Why it matters:** Low click-through performance means the company is paying for exposure but not converting enough attention into traffic or customer interactions.
- **Goal:** Increase CTR, reduce CPC, and bring more qualified users into the sales funnel.

#### 3. Separate campaigns into revenue drivers and efficiency drivers

- **Where to act:** Budget allocation across campaigns.
- **What to do:** Maintain budget for high-revenue campaigns that drive scale, but gradually test more budget on high-ROAS campaigns that are currently underfunded.
- **Why it matters:** High-revenue campaigns help increase sales volume, while high-ROAS campaigns improve marketing efficiency. The company needs both scale and efficiency.
- **Goal:** Balance revenue growth with better budget efficiency.

#### 4. Clarify KPI definitions for better decision-making

- **Where to act:** KPI naming and metric definitions on the dashboard.
- **What to do:** Review the current conversion rate formula. If the metric can exceed 100%, rename it to a more accurate term such as **Sales per Contact**, **Order-to-Interaction Ratio**, or add a clear formula note.
- **Why it matters:** Senior managers need simple and trustworthy KPIs. A conversion rate above 100% may reduce confidence in the dashboard if it is not clearly explained.
- **Goal:** Improve dashboard clarity and make KPI interpretation easier for business users.

---

## 3️⃣ Product & Sales

<img width="1184" height="664" alt="Product and Sales" src="documents/db3.png" />

### Business Questions Answered

- Which product categories and SKUs generate the strongest sales impact from marketing?
- Is marketing budget being allocated to the right product categories?
- Which products should be scaled, optimized, or reduced based on ROAS, spend, revenue, and orders?
- Is product revenue concentrated in only a few top SKUs or spread across many products?

### Key Insights

- **Product revenue is distributed beyond the Top 10 products.**  
  The Top 10 products contributed around **37.7%** of revenue, while other products contributed **62.3%**. This means the company should not only focus on best-selling SKUs, because many non-top products still contribute significant revenue.

- **Váy Chiết Eo Ôm is a high-efficiency category with strong ROAS.**  
  This category generated around **258.05M marketing revenue** with only **11.68M marketing spend**, achieving a strong **Marketing ROAS of 22.09**, much higher than the overall ROAS of **7.67**.

- **Some SKUs inside Váy Chiết Eo Ôm show exceptional efficiency.**  
  Products such as **Kino Dress**, **Lisa Dress**, and **Ginney Dress** show very high ROAS compared with the overall average, suggesting that this category has strong potential for controlled scaling.

- **Core revenue categories still need efficiency control.**  
  Categories such as **Váy Chiết Eo Xòe** and **Áo Tách Set** appear to be important revenue and order drivers, but high revenue categories should still be monitored carefully by ROAS, spend, and profit margin before increasing budget.

### Recommendations

#### 1. Gradually scale high-ROAS product categories

- **Where to act:** High-efficiency categories, especially **Váy Chiết Eo Ôm** and its best-performing SKUs.
- **What to do:** Increase budget gradually by 15–25% for high-ROAS SKUs such as Lisa Dress, Kino Dress, and Ginney Dress, while monitoring ROAS, orders, inventory, and profit margin.
- **Why it matters:** These products generate strong revenue return with relatively low spend, indicating underutilized marketing potential.
- **Goal:** Capture additional revenue from high-efficiency products without reducing ROAS through uncontrolled scaling.

#### 2. Protect core revenue categories but avoid scaling blindly

- **Where to act:** High-revenue and high-order categories such as **Váy Chiết Eo Xòe** and **Áo Tách Set**.
- **What to do:** Maintain their role as core revenue drivers, but evaluate campaign performance by ROAS, profit margin, discount level, inventory, and product-level contribution before increasing budget.
- **Why it matters:** These categories are important for sales volume, but high revenue does not always mean high efficiency or profitability.
- **Goal:** Keep stable sales volume while preventing inefficient budget expansion.

#### 3. Build SKU-level decision groups

- **Where to act:** Product and SKU portfolio.
- **What to do:** Classify SKUs into four groups:  
  - **Hero Products:** high revenue and stable performance  
  - **High-ROAS Products:** strong efficiency and potential to scale  
  - **Optimization Products:** decent sales but weak efficiency  
  - **Low-Priority Products:** low revenue and low ROAS
- **Why it matters:** Product-level marketing performance is not concentrated only in the Top 10 SKUs, so SKU-level grouping helps avoid missing hidden opportunities.
- **Goal:** Make product-level budget decisions more structured and data-driven.

#### 4. Reallocate budget from low-efficiency products to high-efficiency SKUs

- **Where to act:** Product categories or SKUs with high spend but lower ROAS.
- **What to do:** Review product pricing, discount strategy, creative messaging, product page, and audience targeting. If performance does not improve, shift part of the budget to high-ROAS categories and SKUs.
- **Why it matters:** Continuing to spend on low-efficiency products can reduce overall marketing ROAS and limit profitable growth.
- **Goal:** Improve marketing budget efficiency by funding products that generate stronger return per VND spent.
