# Farmer-Market-Insights

# INTRODUCTION

   Agricultural extension specialist plays a vital role in serving as a bridging gap between agricultural research and farmers. They play a critical role in improving agricultural productivity, sustainability and rural development. Often time farmers face challenges like low yields, limited access to the markets, outdated farming methods, lack of modern technologies and facilities, climate adaption and soil health. This analysis helps to explore the relationship between agricultural production and market price trends using SQL based analysis. The focus is to uncover actionable insights by assessing crop profitability, market prices, crop yield evaluation across region by using a dataset containing FarmerAdvisor and MarketResearcher to support better data-driven decision making.
   
# 🧾 OBJECTIVE

  The main objective of this project is to analyze farmer and market data to uncover patterns in price trends, crop profitability, and advisor relationship. 
  Specifically, the goal is to;
1. Calculate average profit per crop type for each farmer.
2. Examine how crops prices vary across different locations
3. Classification of crop based on growth performance.
4. Observe how many farmers fall under each growth category.

# 🛠 TOOLS USED

* Database: MySQL
* CSV Files: Initial data sources.
* Presentation and Report Tools: PowerPoint / Microsoft Word.
* Data Cleaning: Derived missing fields using data analysis techniques like Data manipulation and logical grouping, Derived columns, CTEs, Window functions.

# ✍  Methodology
  The following steps and technique were applied to perform this analysis to generate insights for farmers and market researchers:
  
### 1. Data Preparation

i. Created a new schema “farmer_market_insight

ii. Imported and joined two primary datasets:

    * FarmerAdvisor (contains Farm_ID, Soil_pH, Soil_Moisture, Temperature_C, Rainfall_mm, Crop_Type, Fertilizer_Usage_kg, Pesticide_Usage_kg, Crop_Yield_ton, Sustainability_Score.
    
    * MarketResearcher (contains Market_ID, Product, Market_Price_per_ton, Demand_Index, Supply_Index, Competitor_Price_per_ton, Economic_Indicator, Weather_Impact_Score, Seasonal_Factor, Consumer_Trend_Index.
    
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


# 🔍 KEY INSIGHTS WITH DETAILED RECOMMENDATIONS;
  Here are some of the core findings based on key SQL questions:
  
Q1. Find the top 5 locations where the maximum number of farmers are associated with advisors.

      SELECT farmer_advisor_dataset.Location AS Location, COUNT(*) AS Number_of_farmers
      FROM farmer_advisor_dataset
      GROUP BY Location
      ORDER BY Number_of_farmers
      LIMIT 5;
 <img width="229" height="114" alt="1" src="https://github.com/user-attachments/assets/6b319f62-cf31-417e-998f-f51ba4e6688a" />
 
Insight:
Based on the analyses Location 2 recorded the highest number of farmers associated with advisors, followed closely by location 3, then 4, 5 and 1 respectively.
This suggests effective agricultural extension systems and likely better access to farming guidance.

Recommendation:
•	Invest in logistics and distribution infrastructure in low-profit areas.
•	Consider price regulation policies to support farmers in disadvantaged locations.

Q2. For each crop type, calculate the average market price and sort in descending order.

     SELECT market_researcher_dataset.Product AS Crop_Type, avg(market_researcher_dataset.Market_Price_per_ton) AS Average_Market_Price
         FROM market_researcher_dataset
         GROUP BY 1
         ORDER BY 2 DESC;
         
   <img width="231" height="98" alt="2" src="https://github.com/user-attachments/assets/958358ca-5d97-4f38-ab44-ae69098adb74" />
 
Insights:
The analysis shows that Rice with average market price of 300.840 incurred the highest average market price, followed by corn, soybeans, and wheat. This pricing trend highlights rice as the most valuable crop in the dataset, which indicate significant opportunity for increased farmer income where conditions support its cultivation.

Recommendation
•	Encourage farmers to diversify into higher-price crops where feasible.
•	Provide price trend reports to help farmers time the sale of their produce.

Q3. Count how many unique crops each farmer is associated with.

      SELECT farmer_advisor_dataset.Farm_ID AS Farm_ID, COUNT(DISTINCT farmer_advisor_dataset.Crop_Type) AS Unique_Crops
      FROM farmer_advisor_dataset
      GROUP BY Farm_ID
      ORDER BY Unique_Crops DESC;
   <img width="165" height="254" alt="3" src="https://github.com/user-attachments/assets/a1e91ba2-d279-40f7-af7c-8a7a7ffa945e" />
 
Insight:
 No crop is uniquely associated with any single farmer. This means there’s a high overlap in crop choices, which lead to market saturation and price competition if supply excess demand.
     
 Recommendation:
•	Encourage farmers to grow crops that are in high demand locally or that they grow best, so they can earn more and face less competition.
•	Provide training on multi-crop farming benefits.

Q4. Find farmers who are growing more than 3 different types of crops.

      SELECT farmer_advisor_dataset.Farm_ID AS Farm_ID, COUNT(DISTINCT farmer_advisor_dataset.Crop_Type) As Different_Crop
      FROM farmer_advisor_dataset
      GROUP BY Farm_ID
      HAVING Different_Crop >3;
<img width="191" height="133" alt="4" src="https://github.com/user-attachments/assets/be793dcf-4ee0-4300-b2f2-8681a0cc055d" />

Insight:
All farmers are associated with only one crop type, with no individual growing more than one variety. This indicates a lack of diversification, which creates several potential     risks and lower market price.

Recommendation
•	Introduce education campaigns and demonstration farms to show the benefits of intercropping and crop rotation.
•	Develop localized crop planning guides for diversification.

Q5. List all crops where the max and min market prices (from MarketResearcher) differ by more than ₹10 per kg.

      SELECT market_researcher_dataset.Product AS Crop, 
      MAX(market_researcher_dataset.Market_Price_per_ton)/ 1000 AS Max_Market_Price,
      MIN(market_researcher_dataset.Market_Price_per_ton) / 1000 AS Min_Market_Price,
      (MAX(market_researcher_dataset.Market_Price_per_ton) - MIN(market_researcher_dataset.Market_Price_per_ton)) / 1000 AS Price_Difference
      FROM market_researcher_dataset
      GROUP BY Crop
      ORDER BY Price_Difference >= 10;
   <img width="457" height="107" alt="5" src="https://github.com/user-attachments/assets/563276e3-e6e2-47c2-a772-702e5a8369b8" />
 
 Insight:
Market data indicates that Rice experiences the widest gap between the maximum and minimum market prices, signaling higher price volatility compared to other crops. Wheat, soybeans, and corn shows moderate price variations.

Recommendation
•	Introduce storage and processing solutions to reduce volatility.
•	Provide price forecasting tools to farmers.

Q6. Show all advisors who guide farmers growing the same crop in different districts.

            SELECT farmer_advisor_dataset.Farm_ID, farmer_advisor_dataset.Advisor_ID, COUNT(DISTINCT Crop_Type) AS Crop_Count
            FROM farmer_advisor_dataset
            GROUP BY Farm_ID, Advisor_ID
            HAVING COUNT(DISTINCT Advisor_ID) = 1;
   <img width="228" height="192" alt="6" src="https://github.com/user-attachments/assets/7ed96e1e-535d-4f12-8793-41abe584f196" />
 
Insight:
There is no evidence that all advisor guiding farmer grow the same crop in different districts. This reduces their ability to cope with farm problems like floods, pests or market drops.

Recommendation:
•	Encourage farmers to spread crop production across different locations to reduce the risk of total loss from local issues like pests, bad weather, or price decline.
•	Build regional crop guides based on their insights.

Q7. Rank crops by profit per unit (assume: market price - average cost from FarmerAdvisor) using RANK ().

Insight:
Based on profit per unit (assume: market price - average cost), rice ranks the highest (₹494.14), followed by wheat, corn and soybeans. This highlights rice as a high-value crop.

Recommendation:
•	Promote profitable crops via targeted advisory services: Support rice farming, while helping farmers adopt practices that reduce production risks.
•	Encourage cost management strategies to improve margins.

Q8. Identify locations where the current market price of a crop is more than 20% above the average price of that crop across all locations.

Insight:
No location recorded market prices exceeding 20% above the crop average price, indicating limited regional price advantage and possible lack market channel across zones.

Recommendation:
•	Support farmers near high-price markets with logistics solutions.
•	Promote market-driven production planning.

Q9. List farmers who have been assigned the same advisor for all their crops.
Maintain continuity in advisory assignments for multi-crop farmers.

Insight:
All farmers are consistently linked to the same advisor across all crop records. This shows a stable advisory relationship, which strengthen trust and improved knowledge for a better farm performance.
Recommendation:
•	Introduce tailored support and long-term improvement.
•	Evaluate advisor impact on multi-crop performance.

Q10. Create a CTE showing average profit per crop type by farmer and use it to list farmers making below-average profits on any crop.

Insight:
Several farmers, including Farm_ID 3 (Corn), ID 6 (Rice), ID 2 (Soybean), and ID 1 (Wheat), are earning below average profits; 543,412, 569,472, 547,477, 532,110 respectively. These gaps highlight issues like low yield and market challenges.

Recommendation:
•	Offer targeted training or advisory sessions on low-profit crops.
•	Investigate input cost inefficiencies or pricing issues.

Q11. (Assume MarketResearcher has a PriceDate) — Use a window function to find the price change of each crop over the last 3 entries.

Insight:
Each crop has experienced modest but meaningful price increases:
•	Corn price rose from 100.0710, 100.0858 to 100.1295, marking a +0.0584%. This indicates a relatively stable market with slight upward changes.
•	Rice price climbed from moving from 100.2718, 100.3950 to 100.4308, indicating a +0.1589% increase, which may include heightened demand.
•	Soybean increased from 100.0683, 100.16340 to 100.2554, translating to a +0.1871% rise.
•	Wheat saw the highest shift 100.03762, 100.330225 to 100.5558 at a +0.5182% gain. This growth may reflect improved demand.

Recommendation:
•	Develop dashboards or SMS alerts for real-time price tracking.
•	Help farmers time their sales using trend data.

Q12. Create a new column that classifies crop growth rate as:
    - Low (<20%)
    - Medium (20–50%)
    - High (>50%)
Count the number of crops in each category.

Insight:
Crop growth rates are evenly distributed among Low, Medium, and High categories. This balance may indicate varied crop types, diverse growing conditions and advisory support across regions.
Recommendation:
•	Celebrate high-growth crops in campaigns to promote best practices.
•	Investigate low-growth crops and intervene with soil tests, improved varieties, or irrigation.
•	Promote successful techniques used in high-growth crops to other farming areas.




   
