

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
  
### Q1. Find the top 5 locations where the maximum number of farmers are associated with advisors.

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
 
 Market data indicates that Rice experiences the widest gap between the maximum and minimum market prices, signaling higher price volatility compared to other crops. Wheat, soybeans,     and corn shows moderate price variations.

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

* Encourage farmers to spread crop production across different locations to reduce the risk of total loss from local issues like pests, bad weather, or price decline.

* Build regional crop guides based on their insights.

Q7. Rank crops by profit per unit (assume: market price - average cost from FarmerAdvisor) using RANK ().

Insight:

Based on profit per unit (assume: market price - average cost), rice ranks the highest (₹494.14), followed by wheat, corn and soybeans. This highlights rice as a high-value crop.

Recommendation:

* Promote profitable crops via targeted advisory services: Support rice farming, while helping farmers adopt practices that reduce production risks.
  
* Encourage cost management strategies to improve margins.

Q8. Identify locations where the current market price of a crop is more than 20% above the average price of that crop across all locations.

    WITH Average_Price_Per_Crop AS  (
	SELECT 
		market_researcher_dataset.Product AS Crop, AVG(market_researcher_dataset.Market_Price_per_ton) AS Average_price
	FROM market_researcher_dataset
	GROUP BY Product
    )
    SELECT 
    market_researcher_dataset.Product, market_researcher_dataset.Location, market_researcher_dataset.Market_Price_per_ton,
    ap.Average_price,
    ROUND(((market_researcher_dataset.Market_Price_per_ton - ap.Average_price) / ap.Average_price) * 100, 2) AS Percentage_difference
    FROM market_researcher_dataset
    JOIN Average_Price_Per_Crop ap 
    ON market_researcher_dataset.Product = ap.Crop
    WHERE market_researcher_dataset.Product > ap.Average_price * 1.2;
  <img width="475" height="139" alt="8" src="https://github.com/user-attachments/assets/c27c2a4d-51f7-4e6f-9878-15b1e906891a" />

Insight:

No location recorded market prices exceeding 20% above the crop average price, indicating limited regional price advantage and possible lack market channel across zones.

Recommendation:

* Support farmers near high-price markets with logistics solutions.

* Promote market-driven production planning.

Q9. List farmers who have been assigned the same advisor for all their crops.

      SELECT farmer_advisor_dataset.Farm_ID
      FROM farmer_advisor_dataset
      GROUP BY farmer_advisor_dataset.Farm_ID
      HAVING COUNT(DISTINCT farmer_advisor_dataset.Advisor_ID) = 1;
  <img width="118" height="202" alt="9" src="https://github.com/user-attachments/assets/a73fc046-ff4d-49a2-86cc-5810150c87be" />
  
Insight:

All farmers are consistently linked to the same advisor across all crop records. This shows a stable advisory relationship, which strengthen trust and improved knowledge for a better farm performance.

Recommendation:

* Introduce tailored support and long-term improvement.
  
* Evaluate advisor impact on multi-crop performance.

Q10. Create a CTE showing average profit per crop type by farmer and use it to list farmers making below-average profits on any crop.

Insight:
Several farmers, including Farm_ID 3 (Corn), ID 6 (Rice), ID 2 (Soybean), and ID 1 (Wheat), are earning below average profits; 543,412, 569,472, 547,477, 532,110 respectively. These gaps highlight issues like low yield and market challenges.

Recommendation:
•	Offer targeted training or advisory sessions on low-profit crops.
•	Investigate input cost inefficiencies or pricing issues.

Q11. (Assume MarketResearcher has a PriceDate) — Use a window function to find the price change of each crop over the last 3 entries.

	WITH RankedPrices AS (		
	        SELECT market_researcher_dataset.Product AS Product, market_researcher_dataset.PriceDate AS PriceDate, 
			market_researcher_dataset.Market_Price_per_ton AS Market_price_per_ton,
	       ROW_NUMBER () OVER (PARTITION BY  Product ORDER BY Market_price_per_ton) AS rn
	
	FROM market_researcher_dataset
	)
	SELECT 
	Product, PriceDate, Market_price_per_ton
	FROM RankedPrices
	WHERE rn <=3;

 <img width="297" height="231" alt="11" src="https://github.com/user-attachments/assets/d944f26a-110a-4c0e-bdd2-45dc8be32297" />

Insight:

Each crop has experienced modest but meaningful price increases:

* Corn price rose from 100.0710, 100.0858 to 100.1295, marking a +0.0584%. This indicates a relatively stable market with slight upward changes.

* Rice price climbed from moving from 100.2718, 100.3950 to 100.4308, indicating a +0.1589% increase, which may include heightened demand.

* Soybean increased from 100.0683, 100.16340 to 100.2554, translating to a +0.1871% rise.

* Wheat saw the highest shift 100.03762, 100.330225 to 100.5558 at a +0.5182% gain. This growth may reflect improved demand.

Recommendation:

* Develop dashboards or SMS alerts for real-time price tracking.
* Help farmers time their sales using trend data.

Q12. Create a new column that classifies crop growth rate as:
    - Low (<20%)
    - Medium (20–50%)
    - High (>50%)
Count the number of crops in each category.

         WITH Crop_growth AS (
         SELECT *, (farmer_advisor_dataset.Crop_Yield_ton / AVG(farmer_advisor_dataset.Crop_Yield_ton) OVER()) * 100 AS Crop_growth_rate
         FROM farmer_advisor_dataset
         ),
         Classified_growth AS (
           SELECT Crop_Type, Crop_growth_rate,
              CASE
                 WHEN Crop_growth_rate < 20 THEN 'Low'
                 WHEN Crop_growth_rate BETWEEN 20 AND 50 THEN 'Medium'
                 ELSE 'High'
                 END AS Growth_classification
                 FROM Crop_growth
                 )
         SELECT 
              Growth_classification, COUNT(DISTINCT Crop_Type) AS Number_of_crops
              FROM Classified_growth
              GROUP BY Growth_classification;    

   <img width="253" height="88" alt="1_2" src="https://github.com/user-attachments/assets/5bb9b211-3806-4007-9821-789ad098ba12" />
         
Insight:

Crop growth rates are evenly distributed among Low, Medium, and High categories. This balance may indicate varied crop types, diverse growing conditions and advisory support across regions.

Recommendation:

* Celebrate high-growth crops in campaigns to promote best practices.
  
* Investigate low-growth crops and intervene with soil tests, improved varieties, or irrigation.
  
* Promote successful techniques used in high-growth crops to other farming areas.


Q13. Join the tables and display all farmers whose crop is sold in the same district at the highest price.

		    market_researcher_dataset.Location AS Location,
		    MAX(market_researcher_dataset.Market_Price_per_ton) AS Max_Price
		  FROM market_researcher_dataset
		  GROUP BY Crop, Location)
		SELECT fa.Farm_ID, fa.Crop_Type, fa.Crop_Yield_ton, fa.Location, mr.Market_Price_per_ton
		FROM farmer_advisor_dataset fa
		JOIN market_researcher_dataset mr
		  ON fa.Crop_Type = mr.Product
		  AND fa.Location = mr.Location
		JOIN HighestPricePerCropDistrict mp
		  ON mr.Product = mp.Crop
		  AND mr.Location = mp.Location
		  AND mr.Market_Price_per_ton = mp.Max_Price;

   <img width="438" height="198" alt="13" src="https://github.com/user-attachments/assets/e76733b4-c249-424d-9884-d49145191a2c" />
Insight:

Recommendation:

* Identify and replicate their selling strategies.
* Promote better aggregation or storage options for others.

Q14. For each advisor, count how many of their farmers grow crops that fall under the “High” growth classification (from Q12).

		WITH GrowthRates AS (
			SELECT farmer_advisor_dataset.Farm_ID AS Farm_ID, farmer_advisor_dataset.Crop_Type, farmer_advisor_dataset.Crop_Yield_ton,
			 (farmer_advisor_dataset.Crop_Yield_ton / AVG(farmer_advisor_dataset.Crop_Yield_ton) OVER()) * 100 AS Crop_Growth_Rate
		 FROM farmer_advisor_dataset   
		 ),
		 Classified AS (
		 SELECT *,
		     CASE
		        WHEN Crop_Growth_Rate < 20 THEN 'Low'
		        WHEN Crop_growth_rate BETWEEN 20 AND 50 THEN 'Medium'
		        ELSE 'High'
		        END AS Growth_classification
		        FROM GrowthRates
		 )
		 SELECT COUNT(DISTINCT farm_ID) AS High_Growth_Farmer
		 FROM Classified
		 WHERE Growth_classification = 'High';

 <img width="160" height="56" alt="14" src="https://github.com/user-attachments/assets/9d08e29e-cd0d-41af-9bc8-aed933cd8b52" />
    
Insight:

A total of 8,051 farmers is cultivating crops classified as “High Growth”, which reflects a strong adoption of well-performing crops, productivity, and yield value.

Recommendation:

* Recognize and scale advisor success through peer training.
  
* Assign underperforming farmers to these high-impact advisors.

Q15. Identify if any farmer has duplicate crop entries.
      
      SELECT farmer_advisor_dataset.Farm_ID AS Farmer_ID, farmer_advisor_dataset.Crop_Type AS Crop_Type,
      COUNT(*) AS Duplicate_count
      FROM farmer_advisor_dataset
      GROUP BY Farmer_ID, Crop_Type
      HAVING 3 > 1
      ORDER BY Farmer_ID, Crop_Type;
 <img width="270" height="196" alt="15" src="https://github.com/user-attachments/assets/59a09ab5-21e6-4338-91f8-011fe3852eaf" />

Insight:

No farmer has duplicated crop entries. Each farmer is associated with a distinct crop.

Recommendation:

* Train field staff on accurate digital record-keeping.
  
* Maintain regular data validation checks to ensure accuracy as new records are imputed.

Q16. List all crops grown by farmers that are not listed in the MarketResearcher table.

      SELECT DISTINCT farmer_advisor_dataset.Crop_Type AS Crop_Type
      FROM farmer_advisor_dataset
      LEFT JOIN market_researcher_dataset
      ON farmer_advisor_dataset.Crop_Type = market_researcher_dataset.Product
      WHERE market_researcher_dataset.Product IS NULL
      ORDER BY farmer_advisor_dataset.Crop_Type;
   <img width="142" height="105" alt="16" src="https://github.com/user-attachments/assets/74f32c2b-63c8-447c-a50e-4abb569d8117" />
   
Insight:

No mismatches between what farmers grow and what is recorded in the MarketResearcher table. Farmers are not cultivating any crop that isn’t already listed. This shows strong data integrity, which is important for accurate analysis and reliable decision-making.

Recommendation:
•	Maintain regular data validation checks to ensure accuracy as new records are imputed.
•	Expand market research coverage to include these crops.
•	Educate farmers on marketability before large-scale planting.

Q17. For each crop, determine which location has the highest average profit margin.

      WITH Profit AS (
      SELECT farmer_advisor_dataset.Crop_Type AS Crop, market_researcher_dataset.Location AS Location, 
          AVG(farmer_advisor_dataset.Crop_Yield_ton * market_researcher_dataset.Market_Price_per_ton) AS Average_Profit_margin
      	FROM farmer_advisor_dataset
      	JOIN market_researcher_dataset
      	ON farmer_advisor_dataset.Farm_ID = market_researcher_dataset.Market_ID
      	GROUP BY farmer_advisor_dataset.Crop_Type, market_researcher_dataset.Location
      ),
      RankedProfit AS (
      SELECT *,
          RANK() OVER (PARTITION BY Crop ORDER BY Average_Profit_margin DESC) AS Profit_rank
      FROM Profit)
      SELECT *
      FROM RankedProfit
      WHERE Profit_rank = 1
      ORDER BY Average_Profit_margin DESC;
 <img width="355" height="96" alt="17" src="https://github.com/user-attachments/assets/9908ee88-fcd3-435b-98c6-072d3ec9f26a" />
 
Insight:

Rice (Location 5), Corn in location 4, Wheat (Location 2) and, Soybean (Location 3), shows strong performance in their respective location. These crop location combinations highlight areas with favorable conditions or market access.

Recommendation:

* Prioritize these high performing zones for investment in training, infrastructure, and supply to boost continuous productivity.

* Promote crop zoning and suitability mapping.

Q18. Use window functions to find the second-highest market price crop per district.

      SELECT *
      FROM (
       SELECT market_researcher_dataset.Product AS Crop, market_researcher_dataset.Location AS Location, 
       market_researcher_dataset.Market_Price_per_ton AS Market_price,
       DENSE_RANK() OVER (PARTITION BY market_researcher_dataset.Product, market_researcher_dataset.Location ORDER BY market_researcher_dataset.Market_Price_per_ton DESC
        ) AS Price_rank
      FROM market_researcher_dataset
      ) AS Rank_prices
      WHERE Price_rank = 2
      ORDER BY Market_price desc;
   <img width="303" height="190" alt="18" src="https://github.com/user-attachments/assets/9b6c1340-02e5-4f98-86ed-ea26fd00aa09" />

 Insight:
 
 Based on the analysis, Wheat with the highest market price of 499.9572723, in Location_3 obtained the second highest.

Recommendation:

* Guide farmers to diversify with these crops.
  
* Promote awareness of local market gaps for such crops.

Q19. List all advisors associated with more than 5 distinct crop types.

      SELECT farmer_advisor_dataset.Farm_ID AS Farm_ID, COUNT(DISTINCT farmer_advisor_dataset.Crop_Type) AS Distinct_crop_Type
      FROM farmer_advisor_dataset
      GROUP BY Farm_ID
      HAVING Distinct_crop_Type >=5
   <img width="200" height="122" alt="19" src="https://github.com/user-attachments/assets/cc13216b-126f-4782-8491-a0f9e176c285" />
   
Insight:
No advisor is linked to more than five different crop types, showing a level of specialization. This may result to minimum spread of diverse farming knowledge across advisory networks.

Recommendation:

* Encourage collaboration among advisors to share knowledge and best agricultural practices across different crop types and locations.

* Use their knowledge to develop multi-crop advisory materials.

Q20. Find farmers who consistently grow the same crop type for all season (assume a Season column exists or mock one for the exercise).

      SELECT farmer_advisor_dataset.Farm_ID AS Farm_ID, COUNT(DISTINCT farmer_advisor_dataset.Crop_Type) As Crop_Type
      FROM farmer_advisor_dataset
      GROUP BY  farmer_advisor_dataset.Farm_ID 
      HAVING  COUNT(DISTINCT farmer_advisor_dataset.Crop_Type) = 1
      AND COUNT(DISTINCT farmer_advisor_dataset.Season) >1;
   <img width="176" height="144" alt="20" src="https://github.com/user-attachments/assets/a5ce1f58-50c4-48a6-b9cc-b2c79873a114" />
   
Insight:

Farmers are not growing the same crop in every season, which may indicate poor seasonal planning or no/ limited awareness of rotation benefits. This can may lead to reduced long-term productivity.

Recommendation:

* Introduce seasonal rotation plans with training so that farmers can better manage timing, harvest cycle and soil health.

* Use advisory program to guide farmers in choosing the right crop based on specific season, in other to boost both yield and sustainability.

* Provide incentives to shift toward multi-season cropping.

# FINAL RECOMMENDATIONS

# 1.	Deploy agricultural extension practitioner

* Extension agents conduct individual or group farm visits to asses on site challenges and offer appropriate recommendations suited for soil conditions, farming methods, and specific crops.

* They assist farmers in navigating market trends, minimizing post-harvest losses, securing better prices, to boost income and market competitiveness.

* They organize training sessions, field demonstration, and workshops to equip farmers with modern technique such as pest control, improved seed usage.

# 2.	Crop Rotation Education

One of the key strategies for enhancing sustainable agricultural practices is the promotion of crop rotation education among farmers. Repeated growing of the same crop exhausts specific soil nutrients. Educating farmers helps them understand how alternating crops like leguminous crops can naturally replenish the soil.

Proposed Actions:

* Include crop rotation topics in farmer training and advisory sessions to build awareness of its benefits for sustainability.

* Promote peer learning by encouraging experienced farmers to share success stories and practices with their communities.

* It increases farm resilience to climate variability, helping farmers adapt to environmental changes.

# 3.	Diversification of crop

The adoption of crop diversification significant impact on the roles and responsibilities of both farmer advisor and market researcher., because overreliance on a particular crop type may lead to significant risk from crop failure, and decline market price. As farmers move towards cultivating a wide variety of crops the roles because even more critical in ensuring successful production and informed decision making.

Proposed Actions:

* Include crop diversification in extension training and support research on locally suitable crop combination.

* Equip farmers with knowledge on diversified cropping system which include intercropping, mixed farming, and agroforestry.

* Ensure availability of quality seeds, fertilizers, and market linkages for newly introduced or underutilized crops.

* Offer small grants, farming support, or advice to farmers who want to try new crops or different planting methods.

# 4.	Encourage data collection

Consistent and organized data collection on crop performance, market dynamics and soil health are essential for informed evidence-based decisions by both farmers and market researcher.

Proposed Actions:

* Develop centralized databases to track crop yields, input application, and market prices fluctuations.

* Encourage community involvement in data gathering processes to ensure accuracy.

* Equip farmer advisors and researchers with mobile tools and education for data collections.

# 5.	Promote innovation and sustainability

Encouraging innovation through the introduction of new crops, improved varieties, and sustainable practices fosters resilience in the face of climate change.

Proposed Actions:

* Encourage partnership with research institutions and agro-startups to introduce practical, and locally adaptable innovations.

* Promote incentives for early adopters of sustainable practices, such as certification and access to premium markets.

# CHALLENGES FACED

1. Missing Key Columns: The dataset lacked essential fields like Location, Season, Advisor_ID, and PriceDate, which made direct analysis impossible. I had to manipulate these columns using assumptions, proxies, and logical inferences.
   
2. Inconsistent Farmer Records: Some farmers appeared to switch crops or had multiple entries without clear pattern. This required careful aggregation and consistency checks, especially for profit tracking.
   
3. Unrealistic Date Values: The PriceDate field contained future years like 2036-2049. I resolved this by generating realistic sequential dates using ROW_NUMBER () and DATE_SUB () from a 2024 baseline.
   
# SUMMARY

This analysis explored 20 focused data queries on farmer practices, advisor support, crop diversity, and market dynamics. The results highlight important patterns regions with high advisor engagement tend to have better farmer outcomes, while crop diversification links closely to higher profits and resilience. Price fluctuations in key crops suggest a need for better market intelligence. Some farmers maintain consistent advisory relationships, boosting performance, but data inconsistencies remain a concern. Overall, the insights point to the growing importance of reliable data, targeted advisory services, and informed crop planning in building a more efficient and responsive agricultural system.

# CONCLUSION

Improving agricultural outcomes requires strengthening farmerAdvisor relationships, promoting crop rotation and diversification, and using real-time data. High value crops offer profits but need market access strategies to avoid risk. Filling data gaps and expanding advisory services will support smarter, evidence-based planning. With coordinated action across policy, technology, and local engagement, farmers can achieve better yields, reduced risk, and sustainable livelihoods in a rapidly changing agricultural landscape.


![Thank you](https://github.com/user-attachments/assets/b51cee4e-1004-4c19-99db-98df7b48bdb9)

   
