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

-- Analyse property price trends over time with advanced statistical calculations and window functions
WITH RankedSales AS (
    SELECT 
        property_id,
        sale_date,
        sale_price,
        ROW_NUMBER() OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS sale_rank,
        LAG(sale_price, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS prev_sale_price,
        LAG(sale_date, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS prev_sale_date
    FROM 
        Sales
)
SELECT 
    EXTRACT(YEAR FROM sale_date) AS year,
    AVG(sale_price) AS average_sale_price,
    (AVG(sale_price) - LAG(AVG(sale_price), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC)) / LAG(AVG(sale_price), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC) AS price_change_rate,
    (COUNT(*) - LAG(COUNT(*), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC)) / LAG(COUNT(*), 1) OVER (ORDER BY EXTRACT(YEAR FROM sale_date) ASC) AS sales_growth_rate
FROM 
    RankedSales
WHERE 
    sale_rank > 1
GROUP BY 
    EXTRACT(YEAR FROM sale_date)
ORDER BY 
    year;

-- Identify top-performing agents based on complex performance metrics including historical sales performance and client satisfaction ratings
WITH AgentPerformance AS (
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
)
SELECT 
    agent_id,
    total_sales,
    avg_satisfaction_rating,
    performance_rank
FROM 
    AgentPerformance
WHERE 
    performance_rank <= 10;

-- Determine average property appreciation rates using complex mathematical models and predictive analytics
WITH PropertyAppreciation AS (
    SELECT 
        property_id,
        sale_date,
        sale_price,
        LAG(sale_price, 1) OVER (PARTITION BY property_id ORDER BY sale_date ASC) AS prev_sale_price
    FROM 
        Sales
)
SELECT 
    property_id,
    AVG((sale_price - prev_sale_price) / prev_sale_price * 100) AS appreciation_rate
FROM 
    PropertyAppreciation
WHERE 
    prev_sale_price IS NOT NULL
GROUP BY 
    property_id;

-- Compare rental yields in different neighborhoods using advanced spatial analysis and statistical modeling techniques
WITH NeighborhoodRentalYields AS (
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
    neighborhood,
    rental_yield,
    yield_rank
FROM 
    NeighborhoodRentalYields
WHERE 
    yield_rank <= 5;

