# Dataset superstore_data.csv normalized into 4 main tables for analysis with SQL.

## customers
Basic demographic and customer information.
Important columns:
- Id (PK)
- Year_Birth
- Education
- Marital_Status
- Income
- Kidhome, Teenhome
- Dt_Customer (join date)
## customer_purchases
Total spending per product category per customer.
Columns:
- Id (FK → customers.Id)
- MntWines, MntFruits, MntMeatProducts
- MntFishProducts, MntSweetProducts, MntGoldProds
## customer_behavior
Purchase behavior & visits per channel.
Columns:
- Id (FK)
- NumDealsPurchases, NumWebPurchases
- NumCatalogPurchases, NumStorePurchases
- NumWebVisitsMonth, Recency
## campaign_response
Response to campaigns & complaints.
Columns:
- Id (FK)
- Response (0/1)
- Complain (0/1)
