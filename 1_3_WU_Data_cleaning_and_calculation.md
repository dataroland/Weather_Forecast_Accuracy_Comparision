```python


import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import datetime
import pytz
from bs4 import BeautifulSoup as bs
import requests

IBM = pd.read_csv('/home/dataroland/data_source_course/IBM_weather/all_data/IBM_all_data.csv', delimiter=',')
# read the data
OMSZ = pd.read_csv('/home/dataroland/data_source_course/OMSZ/all_data/OMSZ_all_data.csv', delimiter=',')

IBM = IBM[['actual_date', 'actual_hour', 'version', 'forecast_date', 'forecast_hour', 'temp', 'precip1Hour', 'wind_speed_mph', 'condition']]

IBM['forecast_date'] = pd.to_datetime(IBM['forecast_date'])
IBM['actual_date'] = pd.to_datetime(IBM['actual_date'])
IBM['forecast_hour'] = pd.to_numeric(IBM['forecast_hour'], errors='coerce')
IBM['actual_hour'] = pd.to_numeric(IBM['actual_hour'], errors='coerce').astype(int)
IBM['temp'] = pd.to_numeric(IBM['temp'], errors='coerce').astype(int)
IBM['precip1Hour'] = pd.to_numeric(IBM['precip1Hour'], errors='coerce')
IBM['wind_speed_mph'] = pd.to_numeric(IBM['wind_speed_mph'], errors='coerce')
IBM['version'] = pd.to_numeric(IBM['version'].str.replace('t-', ''), errors='coerce')
today_date = pd.Timestamp(pd.Timestamp.today().date())
IBM = IBM[
    (IBM['forecast_date'] > '2023-06-15') &
    (IBM['forecast_date'] < today_date)
]

IBM_filtered = IBM[(IBM['actual_hour'].isin([0, 6, 12, 18])) | (IBM['version'] == 0)]
IBM_filtered = IBM_filtered.drop(IBM_filtered[(IBM_filtered['forecast_date'] == '2023-10-29') & (IBM_filtered['forecast_hour'] == 2)].index)

time_difference_hours = (IBM_filtered['forecast_date'] - IBM_filtered['actual_date']).dt.total_seconds() / 3600
calculated_value = (((time_difference_hours + IBM_filtered['forecast_hour'] - IBM_filtered['actual_hour']) / 6)).astype(int)
IBM_filtered['version_new'] = calculated_value
IBM_filtered['version_new'] = np.where(
    IBM_filtered['version'] == 0,
    0,
    np.where(
        IBM_filtered['forecast_hour'].isin([0, 6, 12, 18]),
        calculated_value,
        calculated_value + 1
    )
)

IBM_sorted = IBM_filtered.sort_values(by=['forecast_date', 'forecast_hour',  'version_new'], ascending=[True, True, True])

OMSZ = OMSZ[['version', 'forecast_date', 'forecast_hour', 'rain_mm']][OMSZ.version == 't-0']
OMSZ['version_new'] = 0
OMSZ.rename(columns={'rain_mm': 'rain_mm_v2'}, inplace=True)

OMSZ = OMSZ[['forecast_date', 'forecast_hour', 'rain_mm_v2', 'version_new']]
OMSZ['forecast_date'] = pd.to_datetime(OMSZ['forecast_date'])
OMSZ['forecast_hour'] = pd.to_numeric(OMSZ['forecast_hour'])
OMSZ['rain_mm_v2'] = pd.to_numeric(OMSZ['rain_mm_v2'], errors='coerce')

IBM_merged0 = pd.merge(IBM_sorted, OMSZ, on=['forecast_date', 'forecast_hour', 'version_new'], how='left')

IBM_merged0['rain_mm'] = IBM_merged0['precip1Hour'].fillna(0) + IBM_merged0['rain_mm_v2'].fillna(0)

IBM_merged = IBM_merged0[['forecast_date', 'forecast_hour', 'version_new', 'temp', 'rain_mm', 'wind_speed_mph', 'condition']]

def calculate_temp_diff(group):
    # Calculate absolute temperature difference with version 0
    group['temp_diff'] = group['temp'] - group.loc[group['version_new'] == 0, 'temp'].iloc[0]
    group['temp_diff_abs'] = abs(group['temp'] - group.loc[group['version_new'] == 0, 'temp'].iloc[0])
    group['actual_temp'] = group.loc[group['version_new'] == 0, 'temp'].iloc[0]
    group['rain_mm_diff'] = group['rain_mm'] - group.loc[group['version_new'] == 0, 'rain_mm'].iloc[0]
    group['rain_mm_diff_abs'] = abs(group['rain_mm'] - group.loc[group['version_new'] == 0, 'rain_mm'].iloc[0])
    group['actual_rain_mm'] = group.loc[group['version_new'] == 0, 'rain_mm'].iloc[0]
    group['wind_speed_mph_diff'] = group['wind_speed_mph'] - group.loc[group['version_new'] == 0, 'wind_speed_mph'].iloc[0]
    group['wind_speed_mph_diff_abs'] = abs(group['wind_speed_mph'] - group.loc[group['version_new'] == 0, 'wind_speed_mph'].iloc[0])
    group['actual_wind_speed_mph'] = group.loc[group['version_new'] == 0, 'wind_speed_mph'].iloc[0]
    return group

# Apply the function to each group
IBM_merged_analyzed = IBM_merged.groupby((IBM_merged['version_new'] == 0).cumsum()).apply(calculate_temp_diff)

IBM_daily = IBM_merged0[['forecast_date', 'forecast_hour', 'version_new', 'temp', 'rain_mm', 'wind_speed_mph']]
IBM_grouped = IBM_daily.groupby(['forecast_date', 'version_new']).agg({
    'temp': ['min', 'max'],
    'rain_mm': ['sum'],
    'wind_speed_mph': ['mean']
})

IBM_grouped.columns = ['_'.join(col).strip() if col[1] else col[0] for col in IBM_grouped.columns.values]

IBM_grouped = IBM_grouped.reset_index()

def calculate_temp_diff(group):
    # Calculate absolute temperature difference with version 0
    group['temp_max_diff'] = group['temp_max'] - group.loc[group['version_new'] == 0, 'temp_max'].iloc[0]
    group['temp_max_diff_abs'] = abs(group['temp_max'] - group.loc[group['version_new'] == 0, 'temp_max'].iloc[0])
    group['actual_max_temp'] = group.loc[group['version_new'] == 0, 'temp_max'].iloc[0]
    group['temp_min_diff'] = group['temp_min'] - group.loc[group['version_new'] == 0, 'temp_min'].iloc[0]
    group['temp_min_diff_abs'] = abs(group['temp_min'] - group.loc[group['version_new'] == 0, 'temp_min'].iloc[0])
    group['actual_min_temp'] = group.loc[group['version_new'] == 0, 'temp_min'].iloc[0]
    group['rain_mm_diff'] = group['rain_mm_sum'] - group.loc[group['version_new'] == 0, 'rain_mm_sum'].iloc[0]
    group['rain_mm_diff_abs'] = abs(group['rain_mm_sum'] - group.loc[group['version_new'] == 0, 'rain_mm_sum'].iloc[0])
    group['actual_rain_mm'] = group.loc[group['version_new'] == 0, 'rain_mm_sum'].iloc[0]
    group['wind_diff'] = group['wind_speed_mph_mean'] - group.loc[group['version_new'] == 0, 'wind_speed_mph_mean'].iloc[0]
    group['wind_diff_abs'] = abs(group['wind_speed_mph_mean'] - group.loc[group['version_new'] == 0, 'wind_speed_mph_mean'].iloc[0])
    group['actual_wind'] = group.loc[group['version_new'] == 0, 'wind_speed_mph_mean'].iloc[0]
    return group

# Apply the function to each group
IBM_daily_analyzed = IBM_grouped.groupby((IBM_grouped['version_new'] == 0).cumsum()).apply(calculate_temp_diff)

IBM_daily_analyzed_summary = IBM_daily_analyzed[['forecast_date', 'version_new', 'temp_max', 'temp_min', 'rain_mm_sum', 'temp_max_diff', 'temp_max_diff_abs', 'actual_max_temp', 'temp_min_diff', 'temp_min_diff_abs', 'actual_min_temp', 'rain_mm_diff', 'rain_mm_diff_abs', 'actual_rain_mm']]

IBM_daily_analyzed_summary['provider'] = 'WU'

IBM_daily_analyzed_summary = IBM_daily_analyzed_summary[IBM_daily_analyzed_summary['version_new'].isin([4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 44, 48, 52, 56, 60])]

IBM_daily_analyzed_summary['version_new'] = (IBM_daily_analyzed_summary['version_new'] / 4).astype(int)



# Get today's date and time
today_datetime = pd.Timestamp.now()

# Subtract one day to get yesterday's date
yesterday_date = today_datetime - pd.Timedelta(days=1)
formatted_yesterday_date = yesterday_date.strftime('%Y-%m-%d')

IBM_daily_analyzed_summary = IBM_daily_analyzed_summary[IBM_daily_analyzed_summary.forecast_date == formatted_yesterday_date]

IBM_daily_analyzed_summary.to_csv('/home/dataroland/data_source_course/IBM_weather/weather_summary.csv', mode='a', index=False, header=False)

dictionary_path1 = '/home/dataroland/data_source_course/IBM_weather/IBM_upload/'
file_path1 = dictionary_path1 + formatted_yesterday_date
IBM_daily_analyzed_summary.to_csv(file_path1, index=False, header=False)

```
