# Power-BI-Analytics-for-Retail-Fashion-Sales-Stores-Performance  

🧹 Data Cleaning & Preparation Process  

This project uses a messy retail fashion dataset consisting of multiple tables:
sales_data, product_data, customer_data, and store_data.
All data cleaning and preparation were performed using Power Query before building the data model.

1️⃣ Cleaning Customer Data (customer_data)

The following steps were applied to prepare customer data for analysis:

Standardized gender values into:

Male

Female

Other / Unspecified

Replaced invalid values such as "???" under gender with Other / Unspecified

Handled missing email values by replacing them with a generic placeholder(unknown@example.com)

Created Age Groups for customer segmentation:

18–25

26–35

36–45

46–60

60–70

Ensured correct data types for age and categorical fields

This enabled clean demographic analysis and customer segmentation in the report.

2️⃣ Cleaning Product Data (product_data)

The product dataset required multiple transformations due to inconsistent and missing values:

Replaced invalid category values such as "???" with "Unknown"

Replaced missing color values with "Unknown"

Renamed list_price to selling_price for better business clarity

Ensured correct numeric data types for selling_price and cost_price

Identified a data integrity issue where 200 transactions referenced an invalid product_id (P999999), which did not exist in the product table

This issue was not removed at the product level, as it originated from the sales_data

Applied Trim, Clean, and Uppercase transformations on product_id to ensure consistent matching during merge operations

3️⃣ Cleaning Sales Data (sales_data)

The sales dataset required the most extensive cleaning and transformation:

🔹 Handling Missing and Invalid Values

Replaced blank customer_id values with "Unknown"

Replaced null values in the discount column with 0

Applied Trim, Clean, and Uppercase on product_id to avoid merge mismatches

🔹 Handling the P999999 Product Issue (200 Rows)

During the merge with product_data, 200 rows showed empty values for selling_price and cost_price

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

✔ This ensured all 50,000 sales rows matched successfully during merge operations
✔ No sales data was lost

🔹 Merging and Calculated Columns

Merged sales_data with product_data using product_id

Brought in:

selling_price

cost_price

Created new calculated columns in Power Query:

Final Unit Price = selling_price × (1 – discount)

Total Revenue = Final Unit Price × quantity

Total Cost = cost_price × quantity

Profit = Total Revenue – Total Cost

All monetary columns were set to Decimal Number data type to ensure calculation accuracy

4️⃣ Cleaning Store Data (store_data)

The store dataset required minimal cleaning:

Verified unique store_id values

Standardized region names

Ensured correct numeric data type for store size

No major inconsistencies were found

5️⃣ Creating Date Dimension (Dim_Date)

To enable time-based analysis and time intelligence calculations:

Created a dedicated Date Dimension table using DAX

Used CALENDAR() function to generate dates dynamically from the minimum to maximum sales date

Added derived columns:

Year

Month Name

Month Number

Quarter

Marked the table as a Date Table

Created a one-to-many relationship between Dim_Date[Date] and sales_data[date]

This allows accurate YOY, YTD, QTD, and trend analysis.

🧱 Data Modeling Summary

Designed a Star Schema with:

Fact table: sales_data

Dimension tables: product_data, customer_data, store_data, Dim_Date

All relationships are one-to-many and single-direction

This structure ensures:

High performance

Accurate aggregations

Clean slicer behavior

Scalable reporting
