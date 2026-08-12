### 1. Month-over-Month Growth Rate (%Growth/MoM)
```dax
%Growth/MoM = 
VAR CURRENT_SALES_MONTH = [Total_Revenue]
VAR LAST_MONTH = CALCULATE([Total_Revenue], DATEADD(Date_Table[DATE], -1, MONTH))
RETURN
IF(
    ISBLANK(LAST_MONTH) || LAST_MONTH = 0, 
    BLANK(),
    DIVIDE(CURRENT_SALES_MONTH - LAST_MONTH, LAST_MONTH, 0)
)

```
### 2. Average Order Value (AOV)
```DAX
AOV = DIVIDE([Total_Revenue], DISTINCTCOUNT('Fact_Sales'[Backet_Number]), 0)
```
### 3. Basket Size
```DAX
Basket Size = DIVIDE(SUM('Fact_Sales'[OrderQuantity]), DISTINCTCOUNT('Fact_Sales'[Backet_Number]), 0)
```

### 4. Profit margin
```DAX
Margin % = DIVIDE([total_profit],[Total_Revenue],0)
```
### 5. Previous Year
```DAX
Previous Year = CALCULATE([Total_Revenue], SAMEPERIODLASTYEAR(Date_Table[DATE]))
```
### 6. YoY Growth
```DAX
YoY Growth = 
VAR Current_year = [Total_Revenue]
VAR last_year = [Previous Year]
RETURN
IF(
    ISBLANK(last_year) || last_year = 0,
    BLANK(),
    DIVIDE(Current_year - last_year, last_year, 0)
)
```
### 7. Unique Products
```DAX
Unique_Products = 
CALCULATE(
    DISTINCTCOUNT('Fact_Sales'[ProductKey]),
    KEEPFILTERS(NOT(ISBLANK('DimProduct'[ProductKey])))
)
```

## 2. Product Analysis Metrics

### 1. ABC / Pareto Product Classification

> **Overview:** This logic classifies products into **A, B, C** categories based on cumulative profit contribution (80/15/5 Pareto principle) while distinctly isolating **Dead Stock / Non-Profitable** items.

#### Step 1: Cumulative Profit Percentage
```DAX
Profit_Cumulative_% = 
VAR CURRENT_PROF = [total_profit]
VAR CURRENT_KEY = SELECTEDVALUE(DimProduct[ProductKey])   
VAR PROFIT_PER_PRODUCT = CALCULATE([total_profit], ALL(DimProduct))

VAR Cumulative_profit = 
    CALCULATE(
        [total_profit],
        FILTER(
            ALL(DimProduct),
            [total_profit] > CURRENT_PROF || ([total_profit] = CURRENT_PROF && DimProduct[ProductKey] <= CURRENT_KEY)
        )
    )
RETURN
    DIVIDE(Cumulative_profit, PROFIT_PER_PRODUCT, 0)
```

#### Step 2: ABC & Dead Stock Logic
```DAX
ABC_Professional = 
VAR CumulativePercentage = [Profit_Cumulative_%]
VAR ProfitValue = [total_profit]

RETURN
SWITCH(
    TRUE(),
    -- Isolate non-profitable / dead stock items
    ISBLANK(ProfitValue) || ProfitValue <= 0, "Dead Stock",
    
    -- Pareto Cumulative Logic (A: Top 80%, B: Next 15%, C: Remaining 5%)
    CumulativePercentage <= 0.80, "A",
    CumulativePercentage <= 0.95, "B",
    "C"
)
```

### 2. BCG Growth-Share Matrix Classification

> **Overview:** Evaluates product portfolio performance dynamically by categorizing active products into standard **BCG Matrix quadrants** (Stars, Cash Cows, Question Marks, Dogs) relative to company-wide performance baselines, while filtering out non-performing/slow-moving stock.

```DAX
BCG = 
VAR CurrentProductSales = [Total_Revenue]
VAR PriorYearProductSales = [Previous Year]
VAR ProductGrowth = DIVIDE(CurrentProductSales - PriorYearProductSales, PriorYearProductSales, 0)

VAR TotalCompanySales = CALCULATE([Total_Revenue], ALLSELECTED(DimProduct[ProductKey]))
VAR ProductShare = DIVIDE(CurrentProductSales, TotalCompanySales, 0)

VAR TotalProductsCount = 
    CALCULATE(
        DISTINCTCOUNT(DimProduct[ProductKey]), 
        FILTER(
            ALLSELECTED(DimProduct[ProductKey]), 
            [Total_Revenue] > 0
        )
    )
VAR AvgMarketShareLine = DIVIDE(1, TotalProductsCount, 0) 

VAR TotalPriorYearSales = CALCULATE([Previous Year], ALLSELECTED(DimProduct[ProductKey]))
VAR AvgCompanyGrowthLine = DIVIDE(TotalCompanySales - TotalPriorYearSales, TotalPriorYearSales, 0) 

RETURN
IF(
    HASONEVALUE(DimProduct[ProductKey]),
    SWITCH(
        TRUE(),
        -- Exclude zero/negative sales
        ISBLANK(CurrentProductSales) || CurrentProductSales <= 0, "Slow-moving",
        
        ProductGrowth >= AvgCompanyGrowthLine && ProductShare >= AvgMarketShareLine, "Stars (النجوم)",
        ProductGrowth < AvgCompanyGrowthLine && ProductShare >= AvgMarketShareLine, "Cash Cows (البقرة الحلوب)",
        ProductGrowth >= AvgCompanyGrowthLine && ProductShare < AvgMarketShareLine, "Question Marks (علامات استفهام)",
        "Dogs (الكلاب)"
    ),
    BLANK()  
) 
```


### 3. Market Basket Analysis (Co-Purchase Orders)

> **Overview:** Calculates the frequency with which two products are purchased together in the same basket/order using a disconnected product comparison table (`Table_Product_Compare`).

```DAX
Market Basket Analysis = 
VAR OriginalProductOrders = CALCULATETABLE(VALUES(Fact_Sales[Backet_Number]))

VAR OriginalProductName = SELECTEDVALUE(DimProduct[EnglishProductName])

VAR ComparedProductName = SELECTEDVALUE('Table_Product_Compare'[EnglishProductName])

RETURN
IF(
    ISFILTERED(DimProduct[EnglishProductName]) 
    && NOT(ISBLANK(ComparedProductName)) 
    && ComparedProductName <> OriginalProductName, 
    
    CALCULATE(
        DISTINCTCOUNT(Fact_Sales[Backet_Number]),
        OriginalProductOrders,
        DimProduct[EnglishProductName] = ComparedProductName
    ),
    BLANK()
)
```
### 4. Product Purchase Frequency

> **Overview:** Calculates the average number of orders placed per unique customer, helping assess customer retention and product re-order behavior.

```DAX
Product_Purchase_Frequency = 
VAR Total_Orders_Count = DISTINCTCOUNT(Fact_Sales[Backet_Number])
VAR Unique_Customers_Count = DISTINCTCOUNT(Fact_Sales[CustomerKey])
RETURN
    DIVIDE(Total_Orders_Count, Unique_Customers_Count, 0)
```
### 5. Slow-Moving Products Rate (%)

> **Overview:** Measures the percentage of non-performing or dead stock items relative to the total product catalog based on the ABC classification model.

```DAX
Slow_Moving_Products_Rate % = 
VAR TotalProducts = COUNTROWS(DimProduct)

VAR count_Dead_Stock = 
    CALCULATE(
        COUNTROWS(DimProduct),
        DimProduct[تصنيف ABC كعمود] = "Dead Stock"
    )
RETURN
    DIVIDE(count_Dead_Stock, TotalProducts, 0)
```
### 6. Active Products Rate (%)

> **Overview:** Calculates the complement percentage of active/performing products relative to the total product portfolio.

```DAX
Active_Products_Rate % = 1 - [Slow_Moving_Products_Rate %]
```
---

## 3. Customer Analysis Metrics

### 1. RFM Core Metrics (Recency, Frequency, Monetary)

> **Overview:** Calculates the three foundational metrics required for RFM customer segmentation at the individual customer level:
> - **Recency:** Days since the customer's last order relative to the latest company sales date.
> - **Frequency:** Total unique orders placed by the customer.
> - **Monetary:** Total revenue generated by the customer.

#### Frequency
```DAX
Frequency = 
CALCULATE(
    DISTINCTCOUNT(Fact_Sales[Backet_Number]),
    ALLEXCEPT(DimCustomer, DimCustomer[CustomerKey])
)
```

#### Recency
```DAX
Recency = 
VAR max_key_company = 
    CALCULATE(
        MAX(Fact_Sales[OrderDateKey]),
        ALL(Fact_Sales)
    )

VAR max_key_customer = 
    CALCULATE(
        MAX(Fact_Sales[OrderDateKey]),
        ALLEXCEPT(DimCustomer, DimCustomer[CustomerKey])
    )

VAR date_company = 
    IF(
        NOT(ISBLANK(max_key_company)),
        DATE(INT(max_key_company / 10000), INT(MOD(max_key_company, 10000) / 100), MOD(max_key_company, 100)),
        BLANK()
    )

VAR date_customer = 
    IF(
        NOT(ISBLANK(max_key_customer)),
        DATE(INT(max_key_customer / 10000), INT(MOD(max_key_customer, 10000) / 100), MOD(max_key_customer, 100)),
        BLANK()
    )

RETURN
IF(
    ISBLANK(date_customer) || ISBLANK(date_company), 
    BLANK(),
    DATEDIFF(date_customer, date_company, DAY)
)
```

#### Monetary
```DAX
Monetary = 
CALCULATE(
    [Total_Revenue],
    ALLEXCEPT(DimCustomer, DimCustomer[CustomerKey])
)
```


### 2. RFM Scoring Logic (R_Score, F_Score, M_Score)

> **Overview:** Assigns individual scores (1 to 3) for Recency, Frequency, and Monetary values based on statistical percentiles (`PERCENTILE.INC`) and business logic thresholds to categorize customer behavior.

#### Recency Score (R_Score)
```DAX
R_Score = 
VAR R = DimCustomer[Recency]
RETURN
IF(
    R <= PERCENTILE.INC(DimCustomer[Recency], 0.33), 3,
    IF(R <= PERCENTILE.INC(DimCustomer[Recency], 0.66), 2, 1)
)
```

#### Frequency Score (F_Score)
```DAX
F_Score = 
VAR F = DimCustomer[Frequency]
RETURN
IF(
    F >= 6, 3,        
    IF(F >= 2, 2, 1)
)
```

#### Monetary Score (M_Score)
```DAX
M_Score = 
VAR M = DimCustomer[Monetary]
RETURN
IF(
    M >= PERCENTILE.INC(DimCustomer[Monetary], 0.66), 3,
    IF(M >= PERCENTILE.INC(DimCustomer[Monetary], 0.33), 2, 1)
)
```

### 3. Customer Segmentation (RFM Composite Matrix)

> **Overview:** Combines individual R, F, and M scores into a composite string code (e.g., "333", "111") to assign meaningful business segments to each customer, enabling targeted marketing and retention strategies.

```DAX
Customer_Segment = 
VAR total_score = DimCustomer[R_Score] & DimCustomer[F_Score] & DimCustomer[M_Score]
RETURN
    SWITCH(
        total_score,
        "333", "Top Customers",
        "111", "Lost customers",
        "311", "New clients",
        "133", "bought too much, we'll lose them",
        
        "332", "Loyal and active customers",
        "331", "Loyal and active customers",
        "233", "Standard CRM label",
        "211", "Churn-risk customers",
        
        "General customers"  
    )
```
### 4. Dynamic Lost Customers Analysis (Churned Customers)

> **Overview:** Dynamically calculates the count of lost/churned customers as of the current timeline axis date. A customer is defined as lost if they purchased within the preceding 365 days but made no purchases in the most recent 90-day window.

```DAX
Lost_Customers_Category_Fixed = 
VAR CurrentAxisDate = MAX(Date_Table[DATE])

-- Customers active in the prior 365 days
VAR PastCustomers = 
    CALCULATETABLE(
        VALUES(Fact_Sales[CustomerKey]),
        FILTER(
            ALLSELECTED(Date_Table),
            Date_Table[DATE] >= CurrentAxisDate - 365 && Date_Table[DATE] < CurrentAxisDate
        )
    )

-- Customers active in the recent 90-day window
VAR CurrentCustomers = 
    CALCULATETABLE(
        VALUES(Fact_Sales[CustomerKey]),
        FILTER(
            ALLSELECTED(Date_Table),
            Date_Table[DATE] >= CurrentAxisDate - 90 && Date_Table[DATE] <= CurrentAxisDate
        )
    )

RETURN
    COUNTROWS(EXCEPT(PastCustomers, CurrentCustomers))
```

### 5. Category Churn Rate (%)

> **Overview:** Calculates the dynamic churn rate percentage by comparing lost customers against the total active customer base from the preceding 365-day period.

```DAX
Category_Churn_Rate_% = 
VAR CurrentAxisDate = MAX(Date_Table[DATE])

VAR PastCustomersCount = 
    CALCULATE(
        DISTINCTCOUNT(Fact_Sales[CustomerKey]),
        FILTER(
            ALLSELECTED(Date_Table),
            Date_Table[DATE] >= CurrentAxisDate - 365 && Date_Table[DATE] < CurrentAxisDate
        )
    )

VAR LostCustomersCount = [Lost_Customers_Category_Fixed]

RETURN
    DIVIDE(LostCustomersCount, PastCustomersCount, 0)
```
### 6. Dynamic Customer Lifetime Value (CLV)

> **Overview:** Calculates the predicted Customer Lifetime Value dynamically as of the timeline axis date by dividing average revenue per active customer over the prior 365 days by the dynamic churn rate (`Average Revenue Per Customer / Churn Rate`).

```DAX
Dynamic_CLV = 
VAR CurrentAxisDate = MAX(Date_Table[DATE])

-- Total revenue generated over the prior 365 days
VAR PastYearRevenue = 
    CALCULATE(
        [Total_Revenue],
        FILTER(
            ALLSELECTED(Date_Table),
            Date_Table[DATE] >= CurrentAxisDate - 365 && Date_Table[DATE] < CurrentAxisDate
        )
    )

-- Unique active customer count over the prior 365 days
VAR PastCustomersCount = 
    CALCULATE(
        DISTINCTCOUNT(Fact_Sales[CustomerKey]),
        FILTER(
            ALLSELECTED(Date_Table),
            Date_Table[DATE] >= CurrentAxisDate - 365 && Date_Table[DATE] < CurrentAxisDate
        )
    )

VAR AvgRevenuePerCustomer = DIVIDE(PastYearRevenue, PastCustomersCount, 0)
VAR ChurnRate = [Category_Churn_Rate_%]

RETURN
    DIVIDE(AvgRevenuePerCustomer, ChurnRate, 0)
```

