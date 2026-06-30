# Global Remittances Dashboard With Power BI

This repository contains a Power BI project that explores global remittance inflows using data from the World Bank API. The goal is to make complex remittance data easier to understand through interactive visuals and simple storytelling.

## 📊 Dashboard Preview

![Dashboard Landing Page](/Global%20Remittances.png)

[View Interactive Dashboard](https://app.powerbi.com/view?r=eyJrIjoiYWUxNWQyZmQtYjFlYy00ZmM3LTgyOGUtNGMzYTRiODczYTc2IiwidCI6IjRiMmE0YjE5LWQxMzUtNDIwZS04YmIyLWIxY2QyMzg5OThjYyIsImMiOjF9)

## 🌍 Project Overview

The dashboard highlights:
- remittance inflows by country and region
- historical trends over time
- top recipient countries
- comparisons across income levels and lending types

## 🔌 Data Source & API
* **Source:** World Bank Open Data API

    - [Personal Remittances Received](https://api.worldbank.org/v2/country/all/indicator/BX.TRF.PWKR.CD.DT?format=json&per_page=20000)
  
    - [Country Metadata](https://api.worldbank.org/v2/country?format=json&per_page=400)
  
* **Data Type:** Annual remittance inflows by country and region
* **Integration:** Connected directly to the World Bank API using Power Query.
* **Update Frequency:** The project is configured to automatically refresh every Saturday, ensuring the dashboard is updated with the latest remittance data from the World Bank.
  
## 🛠️ Tools Used
- Power BI
- Power Query
- DAX
- Excel

## 📖 How to Use

- Open the Power BI report in Power BI Desktop.
- Refresh the data if needed.
- Use the filters and visuals to explore countries, regions, and trends.


## 🗂️ Data Model in Power BI

The model follows a star-schema style design, with the Remittance Fact Table at the center connected to multiple dimension tables.

Instead of manually building relationships at the report level, the dimension tables were pre-structured in Power Query through merge operations.

The dimension tables were created using Power Query Merge Queries to enrich and normalize the base dataset. The Remittance Fact Table was used as the primary dataset Additional datasets were imported separately to form dimension tables for Region, Income Level, and Lending Type.

Each dimension table was created by performing a Merge Queries operation in Power Query, using the Country column as the common key. After merging, only the relevant columns were expanded to produce clean, structured lookup tables.

![Star Schema](/For%20Data%20Model.JPG)

## 📂 Dataset Overview in Power Query
The dataset was designed using a relational multi-table architecture consisting of seven tables. Five of those are linked, while the remaining two exist without a relation ship to the model. One of the standalone table tracks the last updated date for data governance purposes and the other holds various measures used in this project. 

### Remittance Fact Table 
This table contains the core analytical data:
- Country
- Region
- Year
- Remittance Value

It serves as the primary fact table and connects to all other tables.



### Country Dimension Tables 

Three separate lookup tables were created to enrich the analysis:
- Region Table
  - Column1.id as Country ID
  - Column1.name as Country
  - Column1.region.value as Region
- Income Level Table
  - Country
  - Country ID
  - Income Level
- Lending Type Table
  - Country
  - Country ID
  - Lending Type

Each of these tables connects to the remittance fact table through a common country key to support multi-dimensional filtering.



### Best Year Summary Table
- Country
- Region
- Year
- Remittance Value

This table was created to support the identification and calculation of the peak remittance year per country.


### Last Updated Table
This table tracks the most recent update date of the dataset whenever the data is refreshed. It was intentionally designed to stand alone for this purpose.



## 📐 DAX Measures

Six key measures were created using DAX to support dynamic analysis and aggregation within the Power BI model.



### Total Remittances
Overall sum of remittance.
  ```dax
    Total Remittances = SUM('Remittance Fact Table'[Value])
  ```


### Annual Growth Rate
Year-over-year percentage change.
```dax
Annual Growth Rate = 
VAR CurrentYear = SELECTEDVALUE('Remittance Fact Table'[Year])
VAR CurrentValue = [Total Remittances]
VAR PreviousValue = 
    CALCULATE(
        [Total Remittances],
        FILTER(
            ALL('Remittance Fact Table'[Year]),
            'Remittance Fact Table'[Year] = CurrentYear - 1
        )
    )
    RETURN
    DIVIDE(CurrentValue - PreviousValue, PreviousValue)
```



### Top Recipient Country
The country with the highest total remittance inflow.

```dax
Top Recipient Country = 
VAR TopCountryTable =
    TOPN(
        1,
        ALLSELECTED('Remittance Fact Table'[Country]),
        [Total Remittances],
        DESC
    )
RETURN
    MAXX(
        TopCountryTable,
        'Remittance Fact Table'[Country]
    )
```
### Peak Annual Remittances
The highest recorded remittance value in a single year and corresponding country-year combination.

```dax
Peak Annual Remittance = 
VAR SummaryTable =
    ADDCOLUMNS(
        ALL('Remittance Fact Table'[Year]),
        "TotalAmt", CALCULATE([Total Remittances])
    )

VAR TopRow = 
    TOPN(
        1,
        SummaryTable,
        [TotalAmt], DESC,
        'Remittance Fact Table'[Year], ASC
    )

VAR HighestAmount =
    MAXX(TopRow, [TotalAmt])

VAR BestYear =
    MAXX(TopRow, 'Remittance Fact Table'[Year])

VAR FormattedAmount =
    SWITCH(
        TRUE(),
        HighestAmount >= 1000000000000, "$" & FORMAT(HighestAmount / 1000000000000, "#,##0.0") & "trn",
        HighestAmount >= 1000000000, "$" & FORMAT(HighestAmount / 1000000000, "#,##0") & "bn",
        HighestAmount >= 1000000, "$" & FORMAT(HighestAmount / 1000000, "#,##0") & "M",
        HighestAmount >= 1000, "$" & FORMAT(HighestAmount / 1000, "#,##0") & "K",
        "$" & FORMAT(HighestAmount, "#,##0")
    )
RETURN
FormattedAmount & " (" & BestYear & ")"
```
### Top 10 Remittances
Top 10 countries by total remittance values.

```dax
Top 10 Remittances = 
CALCULATE(
    [Total Remittances],
    TOPN(
        10,
        VALUES('Remittance Fact Table'[Country]),
        [Total Remittances],
        DESC
    )
)
```

### Other Countries Remittances
Remittances from all countries outside the top-performing group.

```dax
Other Countries Remittances = [Total Remittances] - [Top 10 Remittances]
```

## 📊 Dashboard & Visualization

![Dashboard Landing Page](/Global%20Remittances.png)

A single interactive dashboard was developed to enable uswers perform dynamic comparisons and analysis. It incorporates slicers, bookmarks, and buttons.

Three buttons were used with bookmarks to switch between Region, Income Level, and Lending Type slicers. This design lets users toggle between slicers and filter the dashboard based on the dimension they are interested in.

## 📈 Key Insights

- India continues to receive the highest remittance inflows globally.
- Ethiopia recorded the strongest growth in remittance inflows in 2024.
- The top 10 recipient countries account for more than half of total global remittances, indicating significant concentration.
- Europe and Central Asia remains the leading region in total remittance inflows.
- Remittance inflows are primarily concentrated in lower-middle and upper-middle-income countries. However, low-income countries have experienced a decline since the 2022 peak.
- Countries classified under IBRD lending status dominate global remittance inflows.
- Peak remittance years vary significantly across countries, with no single global peak pattern observed.
- 2024 recorded the highest global remittance inflows yet.

## 💡 Recommendations
- Formalize what is still informal: A meaningful share of flows still moves outside regulated channels. Expanding mobile money interoperability and cross-border fintech corridors is more impactful than traditional banking expansion alone.

- Reassess low-income country decline trends seriously: The post-2022 decline in low-income remittances is a warning signal—likely tied to migration barriers, labor market tightening, and cost pressures in host economies.

## 📝 Notes

This project is intended as a portfolio and data storytelling task. It can be expanded with additional visualizations, automated refreshes, or a live deployment in the future.

## 💬 Share Your Feedback

Thank you for exploring this project! Your feedback, suggestions, and ideas for improvement are always welcome.

If you notice any issues, have recommendations for new features, or would like to discuss the project, feel free to:

* Open an issue on this GitHub repository.
* Submit a pull request with improvements.
* Connect with me on LinkedIn to share your thoughts.

Your feedback helps improve the project and makes it more valuable for everyone.

## 👨‍💻 Author
Segun Olakoyenikan | Data Analyst and Storyteller

SQL | Power BI | Advanced Excel | Python

### [Return Home](#global-remittances-data-analysis-using-power-bi)