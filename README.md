# -Real-Estate-Analytics-Platform

## Problem Statement:

This project aims to address the analytical needs of the Sales and Marketing department within a real estate company. The department is facing challenges in efficiently analysing and interpreting vast amounts of real estate data to make informed decisions, such as understanding property price trends, identifying top-performing agents, determining property appreciation rates, and comparing rental yields in different neighborhoods. Due to the complexity and volume of the data, manual analysis is time-consuming and prone to errors, hindering the department's ability to optimize sales strategies, allocate resources effectively, and stay competitive in the market.

## Analysis Plan:

1. **Property Price Trends Analysis:**
    - Utilise historical sales data to analyse property price trends over time.
    - Calculate average sale prices and examine fluctuations across different years.
    - Determine the rate of change in property prices to identify trends and patterns.

2. **Top-Performing Agents Identification:**
    - Analyse sales performance metrics of real estate agents, including total sales volume and client satisfaction ratings.
    - Rank agents based on their sales performance and satisfaction ratings to identify top performers.
    - Provide insights into the strengths and weaknesses of individual agents to optimize resource allocation and training programs.

3. **Property Appreciation Rates Determination:**
    - Assess the appreciation rates of properties using historical sales data.
    - Calculate the percentage change in property prices over time to determine appreciation rates.
    - Identify properties with the highest appreciation rates to guide investment decisions and portfolio management strategies.

4. **Neighborhood Rental Yields Comparison:**
    - Analyse rental yields in different neighborhoods based on property listing data.
    - Calculate the average rental yield for each neighborhood to compare investment potential.
    - Identify neighborhoods with the highest rental yields to target for property acquisition or rental property management initiatives.

```sql

WITH RankedSales AS (
    SELECT 
        property_id,
        sale_date,
        sale_price,
        ROW_NUMBER() OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS sale_rank,
        LAG(sale_price, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS prev_sale_price,
        LAG(sale_date, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS prev_sale_date,
        COUNT(*) OVER (PARTITION BY EXTRACT(YEAR FROM sale_date)) AS total_sales_year,
        AVG(sale_price) OVER (PARTITION BY EXTRACT(YEAR FROM sale_date)) AS avg_sale_price_year
    FROM 
        Sales
),
AgentPerformance AS (
    SELECT 
        agent_id,
        COUNT(*) AS total_sales,
        AVG(client_satisfaction_rating) AS avg_satisfaction_rating,
        ROW_NUMBER() OVER (ORDER BY COUNT(*) DESC, AVG(client_satisfaction_rating) DESC) AS performance_rank
    FROM 
        Sales s
    JOIN 
        Clients c ON s.client_id = c.client_id
    GROUP BY 
        agent_id
),
PropertyAppreciation AS (
    SELECT 
        property_id,
        AVG((sale_price - LAG(sale_price, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC)) / LAG(sale_price, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC) * 100) AS appreciation_rate
    FROM 
        Sales
    GROUP BY 
        property_id
),
NeighborhoodRentalYields AS (
    SELECT 
        neighborhood,
        AVG(listing_price / property_size) AS rental_yield,
        RANK() OVER (ORDER BY AVG(listing_price / property_size) DESC) AS yield_rank
    FROM 
        Properties p
    JOIN 
        Listings l ON p.property_id = l.property_id
    GROUP BY 
        neighborhood
)
SELECT 
    -- Property price trends analysis
    EXTRACT(YEAR FROM sale_date) AS year,
    AVG(sale_price) AS average_sale_price,
    (AVG(sale_price) - LAG(AVG(sale_price), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC)) / LAG(AVG(sale_price), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC) AS price_change_rate,
    (COUNT(*) - LAG(COUNT(*), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC)) / LAG(COUNT(*), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC) AS sales_growth_rate,
    
    -- Top-performing agents analysis
    ap.agent_id AS top_agent_id,
    ap.total_sales AS top_agent_total_sales,
    ap.avg_satisfaction_rating AS top_agent_avg_satisfaction_rating,
    ap.performance_rank AS top_agent_performance_rank,
    
    -- Property appreciation rates analysis
    pa.property_id AS property_id,
    pa.appreciation_rate AS property_appreciation_rate,
    
    -- Neighborhood rental yields analysis
    nry.neighborhood AS neighborhood,
    nry.rental_yield AS neighborhood_rental_yield,
    nry.yield_rank AS neighborhood_yield_rank
FROM 
    RankedSales rs
LEFT JOIN 
    AgentPerformance ap ON rs.sale_date = CURRENT_DATE AND rs.total_sales_year > 100 AND rs.avg_sale_price_year > 1000000
LEFT JOIN 
    PropertyAppreciation pa ON rs.property_id = pa.property_id
LEFT JOIN 
    NeighborhoodRentalYields nry ON rs.sale_rank = 1 AND rs.prev_sale_date = CURRENT_DATE
GROUP BY 
    year, 
    average_sale_price, 
    price_change_rate, 
    sales_growth_rate, 
    top_agent_id, 
    top_agent_total_sales, 
    top_agent_avg_satisfaction_rating, 
    top_agent_performance_rank, 
    property_id, 
    property_appreciation_rate, 
    neighborhood, 
    neighborhood_rental_yield, 
    neighborhood_yield_rank
ORDER BY 
    year DESC;
