# Retail Fashion Sales & Customer Insights Dashboard | Power BI  

## 📖 Project Overview

This project focuses on analyzing messy retail fashion data to uncover insights related to sales performance, product trends, customer behavior, store efficiency, and discount impact.  
The dataset required extensive data cleaning, transformation, and modeling before meaningful insights could be generated. The final output is an interactive Power BI dashboard designed for both executive decision-making and detailed analytical exploration.  

### 🗂️ Dataset  
Source: Kaggle – Retail Fashion Dataset  
Tables used:  
  Sales Data  
  Product Data  
  Customer Data  
  Store Data  
The raw data contained missing values, inconsistent formats, and invalid keys, simulating real-world business data challenges.    

## 🧹 Data Cleaning & Preparation Process  
This project uses a messy retail fashion dataset consisting of multiple tables:  
**sales_data,    
product_data,  
customer_data, and store_data**.  
All data cleaning and preparation were performed using **Power Query** before building the data model.  

### 1️⃣ Cleaning Customer Data (customer_data)  
The following steps were applied to prepare customer data for analysis:  
Standardized gender values into:  
Male  
Female  
Other / Unspecified   

Replaced invalid values such as "???" under gender with **Other / Unspecified**  
Handled missing email values by replacing them with a generic placeholder(unknown@example.com)  
Created **Age Groups** for customer segmentation:  
18–25   
26–35  
36–45  
46–60  
60–70  

Ensured correct data types for age and categorical fields    
This enabled clean demographic analysis and customer segmentation in the report.  

### 2️⃣ Cleaning Product Data (product_data)  

The product dataset required multiple transformations due to inconsistent and missing values:  
Replaced invalid category values such as "???" with **"Unknown"**  
Replaced missing color values with **"Unknown"**  
Renamed list_price to selling_price for better business clarity  
Ensured correct numeric data types for selling_price and cost_price  
Identified a data integrity issue where **200 transactions referenced an invalid product_id (P999999)**, which did not exist in the product table  
This issue was not removed at the product level, as it originated from the sales_data  
Applied **Trim, Clean, and Uppercase** transformations on product_id to ensure consistent matching during merge operations  

### 3️⃣ Cleaning Sales Data (sales_data)  

The sales dataset required the most extensive cleaning and transformation: 

🔹**Handling Missing and Invalid Values**   
Replaced blank customer_id values with **"Unknown"**    
Replaced null values in the discount column with **0**    
Applied **Trim, Clean, and Uppercase** on product_id to avoid merge mismatches  

🔹 **Handling the P999999 Product Issue (200 Rows)**  
During the merge with product_data, **200 rows showed empty values for selling_price and cost_price**  
Investigation revealed that product_id = P999999 existed in sales_data but not in product_data  

To resolve this professionally:
Created a new table called UnknownProductTable using Enter Data  
Added a single row representing a missing product:  
product_id = Unknown_Product  
All attributes = "Unknown"  
selling_price = 0  
cost_price = 0  
Appended this table to product_data  
Replaced P999999 with Unknown_Product in sales_data   

✔ This ensured all **50,000 sales rows matched successfully** during merge operations  
✔ No sales data was lost  

**🔹 Merging and Calculated Columns**  
Merged sales_data with product_data using product_id  
Brought in:  
selling_price  
cost_price  

Created new calculated columns in Power Query:  
**Final Unit Price = selling_price × (1 – discount)  
Total Revenue = Final Unit Price × quantity  
Total Cost = cost_price × quantity   
Profit = Total Revenue – Total Cost**  
All monetary columns were set to **Decimal Number** data type to ensure calculation accuracy  

### 4️⃣ Cleaning Store Data (store_data)  

The store dataset required minimal cleaning:  
Verified unique store_id values  
Standardized region names  
Ensured correct numeric data type for store size  
No major inconsistencies were found  


## 🧱 Data Modeling  

Implemented a Star Schema  
Fact table: sales_data  

Dimension tables:  
  product_data  
  customer_data  
  store_data  
  Dim_Date (created using DAX)    
One-to-many, single-direction relationships for optimal performance  

 ### Creating Date Dimension (Dim_Date)  
To enable time-based analysis and time intelligence calculations:  
Created a dedicated Date Dimension table using DAX  
Used CALENDAR() function to generate dates dynamically from the minimum to maximum sales date  
Added derived columns:  

```dax
Dim_Date = CALENDAR(MIN(sales_data[date]),MAX(sales_data[date]))
```

```dax
Year = YEAR([Date])
```
```dax
Month_Name = FORMAT([Date],"MMMM")
```

```dax
Month_Number = MONTH([Date])
```

```dax
Quarter = "Q" & FORMAT([Date], "Q")
```
```dax
Weekday = FORMAT([Date], "DDDD")
```

```dax
Day = DAY([Date])
```

Marked the table as a Date Table  
Created a one-to-many relationship between Dim_Date[Date] and sales_data[date]  
<img width="1601" height="706" alt="Data Modelling" src="https://github.com/user-attachments/assets/2ea11c09-e38e-45cc-a5ee-0a37388ca5ac" />



### Key Measures & KPIs Used 

```dax
Total Revenue = SUM(sales_data[Total Revenue])
```
```dax
Total Quantity Sold = SUM(sales_data[quantity])
```
```dax
Total Orders = DISTINCTCOUNT(sales_data[transaction_id])
```
```dax
Total Customers = DISTINCTCOUNT(sales_data[customer_id])
```
```dax
Total Cost = SUM(sales_data[Total Cost])
```
```dax
Total Profit = [Total Revenue] - [Total Cost]
```
```dax
Average Discount% = AVERAGE(sales_data[discount])
```
```dax
Profit Margin% = DIVIDE([Total Profit],[Total Revenue])
```
```dax
Average Order Value = DIVIDE([Total Revenue],[Total Orders])
```
```dax
Revenue LY = 
CALCULATE (
    [Total Revenue],
    SAMEPERIODLASTYEAR ( Dim_Date[Date] )
)
```
```dax
Revenue YoY % = 
DIVIDE ( [Total Revenue] - [Revenue LY], [Revenue LY] )
```
```dax
Revenue YTD = 
TOTALYTD ( [Total Revenue], Dim_Date[Date] )
```
```dax
Revenue per Sq Meter = DIVIDE([Total Revenue],SUM(store_data[store_size_m2]))
```
```dax
Repeat Customers = 
CALCULATE (
    DISTINCTCOUNT ( sales_data[customer_id] ),
    FILTER (
        sales_data,
        CALCULATE ( COUNT ( sales_data[transaction_id] ) ) > 1
    )
)
```
```dax
Profit per Product = DIVIDE([Total Profit],DISTINCTCOUNT(product_data[product_id]))
```

# 📊 Report Pages & Visualizations 

## 📊 Page 1 – Executive Sales Overview

**What this page is telling**  

### 1️⃣ What is the purpose of this page? (High-level)  
This page provides a **single-screen summary of overall business performance** for leadership.  
It answers three core executive questions:  
   How much did we sell and earn?  
   Are we growing compared to last year?  
   Which categories, regions, and seasons drive revenue?  

   <img width="1283" height="720" alt="1" src="https://github.com/user-attachments/assets/83485867-9b7a-4911-b4af-a9773906422f" />  

  ### 2️⃣ KPI Cards – Key Business Health Indicators  
🔹 **Total Revenue: $12.41M**  
    Represents total sales generated across all years, regions, and categories.  
    Indicates strong overall business volume.  
    
🔹 **Total Profit: $7.10M** 
    Shows that the business is **highly profitable**, not just revenue-driven.  
    Profit is more than 50% of revenue, indicating healthy margins.  
    
🔹 **Total Quantity Sold: 125K units**  
    Confirms high product movement.  
    Helps distinguish whether revenue is driven by volume or pricing.  
    
🔹 **Average Discount: 5.51%**  
    Indicates controlled discounting.  
    Suggests the company is **not overly dependent on heavy discounts** to drive sales. 
    
🔹 **Revenue YoY %: +25.06%**    
    Strong year-over-year growth.  
    Confirms that revenue is increasing significantly compared to the previous year.  
    This KPI signals **business expansion and positive momentum.**  

👉 **Executive takeaway:**  
The business is growing fast, profitable, and discounting is under control.  

### 3️⃣ Sales Trend by Month – Time-Based Performance  
**What the visual shows:**  
     Monthly revenue trend across the year.  
     Revenue peaks around April–June, with moderate declines toward year-end.  

**Insight:**  
     There is clear seasonality in sales.  
     Mid-year months perform better, possibly due to seasonal demand or campaigns.  
     End-of-year revenue softens, suggesting scope for stronger Q4 promotions.  

**👉 Business value:**  
Helps leadership plan inventory, marketing campaigns, and staffing by season.  

### 4️⃣ Revenue by Product Category – Product Mix Analysis  
**What the Pie chart shows:**  
   Revenue is **evenly distributed across categories**:  
   Accessories, Shoes, Dresses, Tops, Bottoms  
   Each category contributes roughly **~20% of total revenue.**  

**Insight:**  
   No single category dominates revenue.  
   Business risk is **well diversified** across product lines.  
   Indicates balanced product portfolio.  

**👉 Business value:**  
Reduces dependency risk and supports stable long-term growth.  

### 5️⃣ Revenue by Region – Geographic Performance  
**What the bar chart shows:**  
    Lisbon and Algarve are top-performing regions.  
    Coimbra and Porto follow closely, with only marginal differences.  

**Insight:**  
    Regional performance is **well balanced.**  
    No region is significantly underperforming.  
    Indicates consistent market penetration.  

**👉 Business value:**  
Supports confidence in regional expansion strategies and fair resource allocation.  

### 6️⃣ Sales Contribution by Season – Seasonal Strategy  
**What the donut chart shows:**  
   All four seasons contribute almost equally:  
   Summer, Winter, Spring, Fall ≈ 24–25% each.  

**Insight:**  
    Revenue is **not dependent on a single season**.  
    Strong year-round demand.  

**👉 Business value:**  
Helps reduce seasonal risk and supports stable cash flow.  

### 🎯 Key Executive Insights  
The business generated **$12.41M in revenue with strong profitability.**  
Year-over-year growth of **25.06%** indicates rapid expansion.  
Revenue is **well diversified across product categories, regions, and seasons.**  
Discounting remains controlled, protecting margins.  
Clear seasonal patterns exist but do not heavily impact total performance.  

**The Executive Sales Overview page provides a high-level snapshot of business performance, highlighting total revenue, profitability, growth trends, and diversification across products, regions, and seasons. It enables quick decision-making for leadership by combining KPIs with trend and contribution analysis.**  

## 📊 Page 2 – Product Performance Analysis  
**What this page is telling**  

### 1️⃣ Purpose of this page  

This page analyzes **how products perform across categories, sizes, and seasons,** focusing on both **revenue and profitability.**  

It helps answer:  
Which product categories generate the most revenue and profit?  
Which sizes sell the most?  
How does seasonality impact product performance?  
Which individual products are top performers?  

<img width="1281" height="721" alt="2" src="https://github.com/user-attachments/assets/2e9a808b-9c71-4ceb-8e8f-8c5d38b4a68c" />  


### 2️⃣ Revenue by Product Category (Bar Chart)  

**What the visual shows:**  
   Accessories and Shoes are the **top revenue-generating categories** (~$2.48–$2.49M).  
   Dresses, Tops, and Bottoms follow closely.  
   Revenue distribution across categories is **fairly balanced.**  

**Insight:**  
   No single category dominates sales.  
   The business has a **diversified product portfolio**, reducing dependency risk.  

**👉 Business impact:**  
Supports balanced inventory planning and reduces category-level risk.  

### 3️⃣ Sales by Product Size (Bar Chart)  

**What the visual shows:**  
   **Small (S) and Extra Large (XL)** sizes have the highest sales volume.  
   Medium (M) and Large (L) sizes perform slightly lower.  

**Insight:**  
   Demand is skewed toward **S and XL sizes,** not the traditional M/L dominance.  
   Indicates potential customer demographic or fit preferences.  

**👉 Business impact:**  
Helps optimize size-level inventory allocation and reduce stock-outs.  

### 4️⃣ Profit by Category (Donut Chart)    

**What the visual shows:**  
   Profit contribution is evenly distributed across categories.  
   Each category contributes roughly **~20% of total profit.**  
   Total profit shown: **$7.10M.**  

**Insight:**  
   Revenue-heavy categories are also **profit-positive.**  
   No category is driving profit erosion.  

**👉 Business impact:**  
Confirms pricing and discount strategies are consistent across categories.  

### 5️⃣ Category × Season Matrix (Revenue Heatmap)    

**What the matrix shows:**  
   Revenue remains strong across **all seasons.**  
   Slight seasonal shifts:  
   Winter performs marginally better for Accessories.  
   Shoes perform strongly in Spring and Summer.  
   Seasonal totals are almost equal.  

**Insight:**  
   Product categories are **not season-dependent.**  
   Business benefits from year-round demand.  

**👉 Business impact:**  
Supports stable supply chain planning and reduces seasonal volatility.  

### Key Insights Summary   

Revenue and profit are **evenly distributed across product categories.**  
Certain sizes (S and XL) outperform others, guiding inventory strategy.  
Seasonal impact on product performance is minimal.  
A limited set of products drives a significant portion of revenue.  

**The Product Performance Analysis page evaluates revenue and profit across product categories, sizes, and seasons. It identifies top-performing products, size-level demand patterns, and seasonal stability, supporting informed merchandising and inventory planning decisions.**   


## 📊 Page 3 – Customer Insights & Segmentation  
**What this page is telling**  

### 1️⃣ Purpose of this page  
This page focuses on understanding who the customers are and how different customer segments contribute to revenue.  

**It answers:**  
How many unique customers does the business have?  
Which genders and age groups generate more revenue?  
Which cities contribute the most to sales?  
How does customer behavior vary across time and demographics? 

<img width="1284" height="719" alt="3" src="https://github.com/user-attachments/assets/eedf9853-97fb-43b6-a10f-2bcd98d2d617" />  


### 2️⃣ Total Customers KPI  

🔹**Total Customers: 21,276**  
Represents the total number of **unique customers** in the dataset.    
Indicates a **large and diversified customer base**.  

**👉 Business value:**  
Helps leadership understand market reach and customer scale.  

### 3️⃣ Revenue by Customer City (Map Visual)    
**What the map shows:**    
   Revenue concentration across major cities such as **Lisbon, Porto, Coimbra, Faro, and Braga.**    
   Larger bubbles indicate higher revenue contribution.    

**Insight:**  
   Sales are **geographically** concentrated in urban and commercial hubs.  
   Lisbon and Faro emerge as key revenue-driving cities.  

**👉 Business value:**  
Supports targeted regional marketing, store placement, and city-specific promotions.  

### 4️⃣ Revenue by Gender (Bar Chart)  

**What the visual shows:**    

Revenue is fairly balanced across:  
  Male (~ $4.0M)  
  Female (~ $3.8M)  
  Other / Unspecified (~$4.1M)  

**Insight:**  
No single gender group dominates revenue.  
“Other / Unspecified” customers contribute significantly, justifying why these records were retained and labeled rather than removed.  

**👉 Business value:**  
Supports inclusive marketing strategies and validates data-retention decisions.  

### 5️⃣ Revenue by Age Group (Bar Chart)  
**What the visual shows:**    

**46–60 age group** generates the highest revenue (~$3.3M).  

Strong contributions from:  
  36–45 (~ $2.3M)  
  26–35 (~ $2.2M)  

Lower contribution from:  
   60–70 (~$2.0M)  

**Insight:**  

Middle-aged customers (36–60) are the **most valuable segment.**  
Younger customers contribute steadily but at lower levels.  

**👉 Business value:**  
Enables targeted promotions, pricing strategies, and loyalty programs for high-value age groups.  

### 6️⃣ Use of Slicers (Segmentation Power)  

**Slicers included:**  
   Gender  
   Age Group  
   Year  
   City  

**Insight:**  
Allows dynamic exploration of customer behavior.  
Business users can instantly analyze:  
Gender trends by year  
Age group performance by city  
Customer behavior changes over time  

**👉 Business value:**  
Empowers self-service analytics for marketing and sales teams.  

### 🎯 Key Customer Insights  
   The business serves over **21K** unique customers.  
   Revenue is well distributed across genders, indicating broad market appeal.  
   Customers aged **46–60** are the highest revenue contributors.  
   Sales are geographically concentrated in major cities.  
   Customer segmentation enables targeted and data-driven decision-making.  

   **The Customer Insights & Segmentation page analyzes customer demographics and geographic distribution to identify high-value segments. It highlights revenue contribution by gender, age group, and city, enabling targeted marketing and customer-focused business strategies.**  


## 📊 Page 4 – Store & Region Performance   
**What this page is telling**   

### 1️⃣ Purpose of this page  

This page evaluates **how different stores and regions perform,** focusing on:    
  Revenue contribution    
  Regional balance    
  Store efficiency (size vs performance)    
  Monthly regional trends    

It helps answer:    
  Which stores and regions generate the most revenue?    
  Are larger stores always more successful?    
  How does regional performance change over time?    

  <img width="1279" height="723" alt="4" src="https://github.com/user-attachments/assets/14fd74e4-603c-4ec6-96d3-4b2ecaf8d04d" />


### 2️⃣ Revenue by Store (Pie Chart)    

**What the visual shows:**     
  Revenue contribution is evenly distributed across stores:  
  Lisbon Flagship  
  Faro Outlet  
  Coimbra Boutique  
  Porto Center  
  Online store  
  Each store contributes roughly **~20% of total revenue.**  

**Insight:**  
  No single store dominates revenue.  
  Indicates a **well-balanced retail network.**  

**👉 Business value:**    
Reduces dependency risk and suggests consistent store-level performance.   

### 3️⃣ Revenue by Region (Treemap) 

**What the visual shows:**  
Lisbon and Algarve lead regional revenue.  
Coimbra and Porto follow closely.  
Online channel contributes a comparable share.  

**Insight:**   
   Regional performance is **well distributed,** with no major underperforming region.  
   Physical and online channels complement each other.  

**👉 Business value:**  
Supports confidence in regional and omnichannel strategy.  

### 4️⃣ Store Size vs Revenue (Scatter Chart – Key Insight)  

**What the visual shows:**  
  Larger stores do **not always generate higher revenue.**  
  Some medium-sized stores outperform larger ones.  
  Online store achieves high revenue without physical size constraints.  

**Insight:**  
  **Store efficiency matters more than store size.**  
  Revenue per square meter is a more meaningful metric than total size.  

**👉 Business value:**  
Guides decisions on store expansion, optimization, and cost control.  

### 5️⃣ Monthly Revenue Trend by Region (Line Chart)  

**What the visual shows:**  
  Monthly revenue trends vary across regions.  
  Clear seasonal fluctuations exist, but:   
  No region consistently underperforms across the year.  

**Insight:**  
   Regional demand patterns differ slightly by month.  
   Allows identification of peak months for specific regions.  

**👉 Business value:**  
Supports region-specific promotions, staffing, and inventory planning.  

### 🎯 Key Store & Region Insights   
  Revenue is evenly distributed across stores and regions.  
  Larger store size does not guarantee higher revenue.  
  Medium-sized and online stores show strong efficiency.  
  Regional performance remains stable throughout the year.  
  Store efficiency should be prioritized over physical expansion.  

**The Store & Region Performance page analyzes revenue distribution across stores and regions, evaluates store efficiency using size-to-revenue analysis, and highlights monthly regional performance trends to support operational and expansion decisions.**  

## 📊 Page 5 – Discount & Pricing Impact Analysis  
**What this page is telling**    

### 1️⃣ Purpose of this page  

This page analyzes how discounting impacts revenue and profitability, helping the business answer:  
Do higher discounts really increase revenue?  
Which discount levels are profitable?  
Which categories rely more on discounts?  
How discounting behavior changes over time?  
This page is critical for pricing strategy and margin protection.  

<img width="1282" height="715" alt="5" src="https://github.com/user-attachments/assets/412d16c4-27cf-4ba3-b696-83ec39611287" />


### 2️⃣ Discount Impact on Revenue & Profit by Category (Scatter Chart)  
**What the visual shows:**  

Each point represents a product category   
X-axis: Average Discount %  
Y-axis: Total Revenue  
Bubble size: Total Profit  

**Insight:**   
Categories with **slightly higher discounts (~5.6–5.7%)** do not always generate significantly higher revenue.  
Some categories achieve **strong revenue and profit with moderate discounts.**  
Profit bubble sizes vary, showing that **revenue does not always translate into profit.**  

**👉 Business value:**  
Helps identify optimal discount levels and avoid unnecessary margin erosion.  

### 3️⃣ Average Discount % by Category (Donut Chart)      

**What the visual shows:**  
Average discount across categories is tightly clustered around **~5–5.6%.**  
No category is heavily discounted compared to others.    
Overall average discount: **5.51%.**    

**Insight:**  
Discounting strategy is **consistent and controlled** across categories.  
No category relies on aggressive discounting to drive sales.  

**👉 Business value:**  
Confirms disciplined pricing strategy and strong brand positioning.  

### 4️⃣ Profit by Discount Range (Bar Chart – Key Insight)  

**What the visual shows:**    
**0% discount** generates the **highest profit (~$5.1M).**  
Profit declines sharply as discounts increase:  

1–10% → ~$1.3M  
11–20% → ~$0.5M  
21–30% → ~$0.2M  

**Insight:**    

Higher discounts significantly **reduce profitability.**  
Discounting beyond 10% provides diminishing returns.  

**👉 Business value:**  
Strong evidence to **limit high discount campaigns** and focus on value-based pricing.  

### 5️⃣ Discount Trend Over Time (Line Chart)    

**What the visual shows:**    
Discount levels fluctuate slightly throughout the year.  
Mild increase during mid-year and festive months.  
No extreme discount spikes.  

**Insight:**  
Discounting is used **strategically**, likely aligned with campaigns or seasonal demand.  
The business avoids heavy discounting even during peak seasons.  

**👉 Business value:**  
Supports sustainable long-term profitability.  

**6️⃣ Role of Slicers (Controlled Analysis)**  

**Slicers included:**  
Year  
Product Category  
Discount Range  

**Insight:**  

Enables granular analysis, such as:  
Profitability of a category at a specific discount level  
Year-wise discount behavior  
Impact of discount range on revenue and profit  

**👉 Business value:**  
Supports data-driven pricing and promotional decisions.

### 🎯 Key Discount & Pricing Insights 

Higher discounts do not guarantee higher revenue.  
Zero or low discount sales generate the highest profit.  
Discounting above 10% significantly erodes margins.  
Average discount levels are consistent and well-controlled.  
Discount strategy appears intentional rather than reactive.  

**The Discount & Pricing Impact Analysis page examines the relationship between discount levels, revenue, and profitability. It highlights that higher discounts significantly reduce margins and that most profit is generated at low or zero discount levels, supporting data-driven pricing strategies.**  

## 🏁 Final Project Conclusion  

This project demonstrates an **end-to-end Business Intelligence workflow,** transforming messy retail fashion data into **actionable business insights** using Power BI.  

Starting from raw datasets with missing values, inconsistent formats, and invalid references, the focus was placed on **data cleaning, validation, and modeling** to ensure accuracy and reliability. A proper **star schema** was implemented, supported by a custom Date dimension and robust DAX measures for revenue, profitability, and time-based analysis.  

Through five focused report pages, the analysis delivered insights across:   
👉Overall business performance and growth trends   
👉Product and category-level performance   
👉Customer demographics and segmentation  
👉Store and regional efficiency  
👉Discount and pricing impact on revenue and profit    

The dashboards emphasize **business decision-making,** revealing that:  
👉Revenue and profit are well balanced across products, regions, and seasons  
👉Business growth is strong and sustainable without heavy reliance on discounts  
👉Certain customer segments and product sizes contribute disproportionately to revenue  
👉Store efficiency matters more than physical size  
👉High discounts significantly erode profitability  

This project reflects **real-world BI challenges**, where imperfect data must be managed rather than removed, and insights must be clearly communicated to both executive and operational stakeholders. Overall, it showcases practical skills in **data analytics, business reasoning, and dashboard storytelling**, making it suitable for real-world analytics roles.  

<img width="360" height="217" alt="Thank you" src="https://github.com/user-attachments/assets/1101e0f9-7ea2-403c-971e-1de966c9ac57" />


