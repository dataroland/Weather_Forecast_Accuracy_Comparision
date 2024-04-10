# Data Gathering, Cleaning, and Processing

## 1) Data Collection:

Data has been gathered from three weather forecast providers: OMSZ, Weather Underground, and Idokep, utilizing their respective websites. The following section provides detailed information on the types of data collected, the frequency of downloads, and other relevant specifics.: 

![Weather_summary](https://github.com/dataroland/Weather_Forecast_Accuracy_Comparision/assets/145594847/22e4f32a-4728-446a-95a8-d640374887f2)

## 2) Data Cleaning and Calculation:

A significant challenge was to identify and compute comparable metrics that are available from all forecast providers. The key indicators established are: 
- High Temperature Accuracy: Percentage of daily maximum temperature forecasts within a 3°C range, averaged over 1 to 6 days.
- Low Temperature Accuracy: Percentage of daily minimum temperature forecasts within a 3°C range, averaged over 1 to 6 days.
- Rain Accuracy: Percentage of correctly forecasted days with more than zero mm of rain, averaged over 1 to 6 days. (Final calculation in Power BI report.)
- No Rain Accuracy: Percentage of correctly forecasted days with zero mm of rain, averaged over 1 to 6 days. (Final calculation in Power BI report.)

These indicators were derived through a multi-step process:
- Calculating different forecast versions and converting them into daily data.
- Determining the discrepancy between the forecasted data and actual weather outcomes.
- Computing the final metrics for all the providers.

This elaborate process underpins the Power BI presentation and website features, providing a thorough analysis for the accurate comparison of the three weather forecast providers. The dataset is updated daily with the previous day's weather data, ensuring the most current information is available.
