# Data Gathering, Cleaning, and Processing

## 1) Data Collection:

I have collected data from 3 different weather forecast providers (OMSZ, Weather Underground and Idokep). The type of data, the frequency of the downloads and other details can be found in the picture below: 

![Weather_summary](https://github.com/dataroland/Weather_Forecast_Accuracy_Comparision/assets/145594847/22e4f32a-4728-446a-95a8-d640374887f2)

## 2) Data Cleaning:

The subsequent step involves refining the data through a SQL script to ensure a focus on pertinent information for analysis. Exclusions encompass:

- **Train Stations:** exclusion of local or foreign stations out of the analyzed line (approximately 27% of all data).
- **Intermediate and Technical Stations:** exclusion of stations within the train line that are not universally served by all trains, along with technical stations (approximately 5% of all data).
- **Incomplete Train Journeys and Excessive Delays:** exclusion of trains unable to complete the line, those missing actual arrival or departure times at any of the stations, or experiencing delays longer than 150 minutes (approximately 4% of all data).

Additionally, a station mapping table is prepared to facilitate operations, mapping train lines with analyzed stations and marking start, end, middle, and out-of-scope stations.

## 3) Data Calculation:

Within the script, crucial features are calculated from the collected data:

- **Delay calculation:**
  - Arrival time difference (in minutes) between actual and timetabled.
  - Departure time difference (in minutes) between actual and timetabled.
- **Delay Components:**
  - Running delay (delay between two stations or faster travel).
  - Stop delay (waiting longer than timetabled).
  - Station delay (combination of running delay and stop delay).
- **Additional Features:**
  - Week number.
  - Month number.
  - Distance in kilometers (excluding 'km').
  - Day of the week.

This comprehensive script serves as the foundation for the Power BI presentation, prediction models, and website functionalities, ensuring detailed insights for each of the specified train routes. The dataset is refreshed on a daily basis with the previous day's delay data.
