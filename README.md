# Farmer-Market-Insights

# INTRODUCTION

   Agricultural extension specialist plays a vital role in serving as a bridging gap between agricultural research and farmers. They play a critical role in improving agricultural productivity, sustainability and rural development. Often time farmers face challenges like low yields, limited access to the markets, outdated farming methods, lack of modern technologies and facilities, climate adaption and soil health. This analysis helps to explore the relationship between agricultural production and market price trends using SQL based analysis. The focus is to uncover actionable insights by assessing crop profitability, market prices, crop yield evaluation across region by using a dataset containing FarmerAdvisor and MarketResearcher to support better data-driven decision making.
   
# OBJECTIVE

  The main objective of this project is to analyze farmer and market data to uncover patterns in price trends, crop profitability, and advisor relationship. 
  Specifically, the goal is to;
1. Calculate average profit per crop type for each farmer.
2. Examine how crops prices vary across different locations
3. Classification of crop based on growth performance.
4. Observe how many farmers fall under each growth category.

# TOOLS USED

* Database: MySQL
* CSV Files: Initial data sources.
* Presentation and Report Tools: PowerPoint / Microsoft Word.
* Data Cleaning: Derived missing fields using data analysis techniques like Data manipulation and logical grouping, Derived columns, CTEs, Window functions.

# Methodology
  The following steps and technique were applied to perform this analysis to generate insights for farmers and market researchers:
  
### 1. Data Preparation

i. Created a new schema “farmer_market_insight

ii. Imported and joined two primary datasets:

    * FarmerAdvisor (contains Farm_ID, Soil_pH, Soil_Moisture, Temperature_C, Rainfall_mm, Crop_Type, Fertilizer_Usage_kg, Pesticide_Usage_kg, Crop_Yield_ton, Sustainability_Score.
    
    * MarketResearcher (contains Market_ID, Product, Market_Price_per_ton, Demand_Index, Supply_Index, Competitor_Price_per_ton, Economic_Indicator, Weather_Impact_Score,     Seasonal_Factor, Consumer_Trend_Index.
    
iii. Ensured consistency in all columns, checked for null or missing values, and aligned crop types and farm ID accurate JOIN operations.    

### 2. Data Derivation

i. Location column (Districts):
The main dataset did not include a direct Location or District column. Since the data lacks actual geotags or textural hints, so I assigned a location or district using CASE statement based on ranges of Farm_ID.

* This was done using this logic
  
      CASE
      WHEN Farm_ID BETWEEN 1 AND 2000 THEN ‘Location_1
      WHEN Farm_ID BETWEEN 2001 AND 4000 THEN ‘Location_2
      WHEN Farm_ID BETWEEN 4001 AND 6000 THEN ‘Location_3
      WHEN Farm_ID BETWEEN 6001 AND 8000 THEN ‘Location_4
      ELSE ‘Location_5
      END AS Location
      FROM farmer_advisor_dataset;
  
* The same logic was applied to the market_researcher_dataset to allow interlinking. This allows for location-based insights.

 ii. PriceDate column:
  
  There was no time-related PriceDate in the MarketResearcher dataset. To enable time series analysis and trend calculations I created a new PriceDate column by assigning synthetic         dates to each record.
  
    This was done using this logic:
  
       ALTER TABLE market_researcher_dataset
       ADD COLUMN PriceDate DATE;
       UPDATE market_researcher_dataset
       SET PriceDate = DATE_SUB (‘2025-01-01’), INTERVAL Market_ID DAY);
  
   Here, id represent the primary key, simulating daily interval from a base date (2025-01-01).

iii. Advisor_ID:  

The dataset did not include an Advisor_ID field. However, each Farm_ID associated with the same advisory behavior. So, I assumed that each farm operates under a fixed advisory model, so I used Farm_ID as a stand-in for Advisor_ID.
This approach allowed evaluating advisory consistency and its impact on crop performance across various crops.

iv. Season column: 

Since the dataset lacks an explicit season field, I derived it using temperature and rainfall data from the FarmerAdvisor table.

The classification was guided by common agronomic patterns:

•	Dry season: characterized by high temperature and low rainfall

•	Mid-season: Characterized by moderate temperature and rainfall

•	Rainy season: Characterized by low temperature and rainfall.

•	Unknown: Conditions that do not fit the above thresholds.

I began by adding a new season column:

                  ALTER TABLE FarmerAdvisor
                  ADD COLUMN Season VARCHAR (50);
                  Then we updated the column based on climate thresholds:
                  UPDATE FarmerAdvisor
                  SET Season = CASE
                            WHEN Temperature >30 AND Rainfall < 50 THEN ‘Dry’
                            WHEN Temperature BETWEEN 20 AND 30 
                           AND Rainfall BETWEEN 50 AND 150 THEN ‘Mid-Season’                                                        
                            WHEN Temperature <20 AND Rainfall >150 THEN ‘Rainy’
                            ELSE ‘Unknown’
                               END;
                               
### 3. Used SQL Techniques (CTEs, JOINS, Window Functions)

•	CTEs (Common Table Expressions) This helps in breaking down complex logic into readable chunks. Example, AvgProfitPerCrop, HighGrowthRate, HighestPricePerDistricts.

•	JOINs: interlinked rows across FarmerAdvisor and MarketResearcher for accurate analysis. joined tables using shared fields like Farm_ID, Crop_Type and location where applicable).

•	Window Functions: Applied RANK (), ROW NUMBER (), DENSE_RANK ()) like: Ranking crops by market price per district, Finding the top and second-highest crop price per location, analyzing crop price trends over time (curated a PriceDate column).



   
