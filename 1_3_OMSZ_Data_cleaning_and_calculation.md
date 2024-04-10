```python

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import datetime
import pytz
from bs4 import BeautifulSoup as bs
import requests


# read the data
OMSZ = pd.read_csv('file_path', delimiter=',')

# change temperature to numeric data
OMSZ['temp'] = pd.to_numeric(OMSZ['temp'], errors='coerce')
OMSZ['rain_mm'] = pd.to_numeric(OMSZ['rain_mm'], errors='coerce')
OMSZ['wind_kmh'] = pd.to_numeric(OMSZ['wind_kmh'], errors='coerce')
OMSZ['wind_max_kmh'] = pd.to_numeric(OMSZ['wind_max_kmh'], errors='coerce')
OMSZ['pressure'] = pd.to_numeric(OMSZ['pressure'], errors='coerce')
OMSZ['temp_max'] = pd.to_numeric(OMSZ['temp_max'], errors='coerce')
OMSZ['temp_min'] = pd.to_numeric(OMSZ['temp_min'], errors='coerce')

# change actual_datetime to datetime
OMSZ['actual_datetime'] = pd.to_datetime(OMSZ['actual_datetime'])
OMSZ['actual_date'] = pd.to_datetime(OMSZ['actual_date'])
OMSZ['forecast_date'] = pd.to_datetime(OMSZ['forecast_date'])

# create version_new column
time_difference_hours = (OMSZ['forecast_date'] - OMSZ['actual_date']).dt.total_seconds() / 3600
calculated_value = ((time_difference_hours + OMSZ['forecast_hour'] - OMSZ['actual_hour']) / 6).round(0).astype(int)

OMSZ['version_new'] = np.where(
    OMSZ['version'] == 't-0',
    0,
    np.where(
        calculated_value == 0,
        1000,
        calculated_value
    )
)

# Filter only the rows where there are actual and forecast data:

conditions = [
    ((OMSZ['forecast_hour'] == 1) & (OMSZ['forecast_date'] > '2023-10-29')) | ((OMSZ['forecast_hour'] == 2) & (OMSZ['forecast_date'] <= '2023-10-29')),
    ((OMSZ['forecast_hour'] == 7) & (OMSZ['forecast_date'] > '2023-10-29')) | ((OMSZ['forecast_hour'] == 8) & (OMSZ['forecast_date'] <= '2023-10-29')),
    ((OMSZ['forecast_hour'] == 13) & (OMSZ['forecast_date'] > '2023-10-29')) | ((OMSZ['forecast_hour'] == 14) & (OMSZ['forecast_date'] <= '2023-10-29')),
    ((OMSZ['forecast_hour'] == 19) & (OMSZ['forecast_date'] > '2023-10-29')) | ((OMSZ['forecast_hour'] == 20) & (OMSZ['forecast_date'] <= '2023-10-29'))
]

# Define the choices corresponding to the conditions
choices = ['night', 'morning', 'noon', 'evening']

# Use numpy's select function to apply the conditions and choices
OMSZ['analyzed_version'] = np.select(conditions, choices, default='not_analyzed')

# Additionally filter by date and version
today_date = pd.Timestamp(pd.Timestamp.today().date())
OMSZ_analyzed = OMSZ[
    (OMSZ['analyzed_version'] != 'not_analyzed') &
    (OMSZ['forecast_date'] > '2023-06-15') &
    (OMSZ['forecast_date'] < today_date) &
    (~OMSZ['version_new'].isin([1000, -2]))
]

OMSZ_analyzed = OMSZ_analyzed.sort_values(by=['forecast_date', 'forecast_hour', 'version_new'], ascending=[True, True, True])

OMSZ_analyzed_new = OMSZ_analyzed.sort_values(by=['forecast_date', 'forecast_hour', 'actual_datetime'], ascending=[True, True, False])
def calculate_temp_diff(group):
    # Calculate absolute temperature difference with version 0
    group['temp_diff'] = group['temp'] - group.loc[group['version_new'] == 0, 'temp'].iloc[0]
    group['pressure_diff'] = group['pressure'] - group.loc[group['version_new'] == 0, 'pressure'].iloc[0]
    group['temp_diff_abs'] = abs(group['temp'] - group.loc[group['version_new'] == 0, 'temp'].iloc[0])
    group['pressure_diff_abs'] = abs(group['pressure'] - group.loc[group['version_new'] == 0, 'pressure'].iloc[0])
    group['actual_temp'] = group.loc[group['version_new'] == 0, 'temp'].iloc[0]
    group['actual_pressure'] = group.loc[group['version_new'] == 0, 'pressure'].iloc[0]
    return group

# Apply the function to each group
OMSZ_analyzed_new = OMSZ_analyzed_new.groupby((OMSZ_analyzed_new['version_new'] == 0).cumsum()).apply(calculate_temp_diff)
OMSZ_analyzed_new1 = OMSZ_analyzed_new[['actual_datetime', 'actual_date', 'actual_hour', 'forecast_date', 'forecast_hour', 'temp', 'pressure', 'version_new', 'analyzed_version', 'temp_diff', 'pressure_diff', 'temp_diff_abs', 'pressure_diff_abs', 'actual_temp', 'actual_pressure']]

OMSZ_analyzed2 = OMSZ[
    (OMSZ['forecast_date'] > '2023-06-15') &
    (OMSZ['forecast_date'] < today_date) &
    (~OMSZ['version_new'].isin([1000, -2]))
]

OMSZ_analyzed_new2 = OMSZ_analyzed2[['forecast_date', 'version_new', 'temp_max', 'temp_min', 'rain_mm', 'wind_kmh', 'wind_max_kmh']].groupby(['forecast_date', 'version_new']).agg({
    'temp_max': 'max', 
    'temp_min': 'min',
    'rain_mm': 'sum',
    'wind_kmh': 'mean',
    'wind_max_kmh': 'mean'
})
OMSZ_analyzed_new2 = OMSZ_analyzed_new2.reset_index()
def calculate_temp_diff2(group):
    # Calculate absolute temperature difference with version 0
    group['temp_max_diff'] = group['temp_max'] - group.loc[group['version_new'] == 0, 'temp_max'].iloc[0]
    group['temp_min_diff'] = group['temp_min'] - group.loc[group['version_new'] == 0, 'temp_min'].iloc[0]
    group['rain_mm_diff'] = (group['rain_mm'] - group.loc[group['version_new'] == 0, 'rain_mm'].iloc[0]).round(1)
    group['wind_kmh_diff'] = group['wind_kmh'] - group.loc[group['version_new'] == 0, 'wind_kmh'].iloc[0]
    group['wind_max_kmh_diff'] = group['wind_max_kmh'] - group.loc[group['version_new'] == 0, 'wind_max_kmh'].iloc[0]
    group['temp_max_abs_diff'] = abs(group['temp_max'] - group.loc[group['version_new'] == 0, 'temp_max'].iloc[0])
    group['temp_min_abs_diff'] = abs(group['temp_min'] - group.loc[group['version_new'] == 0, 'temp_min'].iloc[0])
    group['rain_mm_abs_diff'] = abs(group['rain_mm'] - group.loc[group['version_new'] == 0, 'rain_mm'].iloc[0]).round(1)
    group['wind_kmh_abs_diff'] = abs(group['wind_kmh'] - group.loc[group['version_new'] == 0, 'wind_kmh'].iloc[0])
    group['wind_max_kmh_abs_diff'] = abs(group['wind_max_kmh'] - group.loc[group['version_new'] == 0, 'wind_max_kmh'].iloc[0])
    group['temp_max_actual'] = group.loc[group['version_new'] == 0, 'temp_max'].iloc[0]
    group['temp_min_actual'] = group.loc[group['version_new'] == 0, 'temp_min'].iloc[0]
    group['rain_mm_actual'] = group.loc[group['version_new'] == 0, 'rain_mm'].iloc[0]
    group['wind_kmh_actual'] = group.loc[group['version_new'] == 0, 'wind_kmh'].iloc[0]
    group['wind_max_kmh_actual'] = group.loc[group['version_new'] == 0, 'wind_max_kmh'].iloc[0]
    return group

# Apply the function to each group
OMSZ_analyzed_new3 = OMSZ_analyzed_new2.groupby((OMSZ_analyzed_new2[ 'version_new'] == 0).cumsum()).apply(calculate_temp_diff2)

OMSZ_analyzed_summary = OMSZ_analyzed_new3[['forecast_date', 'version_new', 'temp_max', 'temp_min', 'rain_mm', 'temp_max_diff', 'temp_max_abs_diff', 'temp_max_actual', 'temp_min_diff', 'temp_min_abs_diff', 'temp_min_actual', 'rain_mm_diff', 'rain_mm_abs_diff', 'rain_mm_actual']]

OMSZ_analyzed_summary['provider'] = 'omsz'

OMSZ_analyzed_summary = OMSZ_analyzed_summary[OMSZ_analyzed_summary['version_new'].isin([4, 8, 12, 16, 20, 24])]

OMSZ_analyzed_summary['version_new'] = (OMSZ_analyzed_summary['version_new'] / 4).astype(int)

# Get today's date and time
today_datetime = pd.Timestamp.now()

# Subtract one day to get yesterday's date
yesterday_date = today_datetime - pd.Timedelta(days=1)
formatted_yesterday_date = yesterday_date.strftime('%Y-%m-%d')

OMSZ_analyzed_summary = OMSZ_analyzed_summary[OMSZ_analyzed_summary.forecast_date == formatted_yesterday_date]

OMSZ_analyzed_summary.to_csv('file_path', mode='a', index=False, header=False)

dictionary_path1 = 'file_path'
file_path1 = dictionary_path1 + formatted_yesterday_date
OMSZ_analyzed_summary.to_csv(file_path1, index=False, header=False)

```
