# Power BI Presentation:

For visualizing train delay data, I opted for [Microsoft Power BI](https://powerbi.microsoft.com/), an interactive data visualization software developed by Microsoft, primarily designed for business intelligence. Power BI comprises a suite of software services, apps, and connectors that collaborate seamlessly to transform diverse data sources into both static and interactive visualizations. Data can be sourced from databases, web pages, PDFs, or structured files such as spreadsheets, CSV, XML, JSON, XLSX, and SharePoint.

## 1) Data Discovery / Data Shaping / Data Modeling:

I integrated Power BI with my PostgreSQL database on my Ubuntu server importing SQL tables and direct queries. Daily, the SQL table receives new data, representing the previous day's weather information, automatically reflecting in the reports after refreshing. Several transformations were applied to the data:

- Changing column types
- Adding columns like grouping forecast errors
- Calculating measures like 'rain' and 'no rain' forecast accuracy  
- I established connections to historical weather data table through date columns (many-to-one relationship).

## 2) Data Visualization:

Diverse charts were crafted to represent weather forecast accuracy data, including bar and line charts, histograms, simple tables.

- Download statistics, displaying daily data received
- High and Low Temperature forecast accuracy in percentage and in Celsius
- 15+ days High Temperature forecast accuracy vs weather historical data
- Forecast error chart for High Temperature forecasts
- 5+ Celsius Change of High Temperature Forecast Accuracy 
- Rain and No Rain forecast accuracy in percentage

## 3) Data Sharing:

This report serves a personal purpose, designed exclusively for my use without publication or sharing with others.
