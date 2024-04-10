```python

#Daily_data

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import datetime
import pytz
from bs4 import BeautifulSoup as bs
import requests

## download forecast data

url = "https://www.idokep.hu/30napos/Budapest"
response = requests.get(url)
html = response.content
soup_forecast = bs(html, "lxml")

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)
i = 1
j = 0
forecast_data_full = []
for div in soup_forecast.find_all("div", class_="ik dailyForecastCol"):
    forecast_data = {}
    forecast_data['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
    forecast_data['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
    forecast_data['actual_day'] = converted_datetime.strftime("%d")
    forecast_data['actual_hour'] = converted_datetime.strftime("%H")
    forecast_data['version'] = "t-" + str(i)
    i = i + 1
    forecast_data['forecast_date'] = (converted_datetime + datetime.timedelta(days=j)).strftime("%Y-%m-%d")
    j = j + 1
    forecast_data['forecast_day'] = div.find(class_="ik dfColHeader").span.get_text(strip=True)
    forecast_data['condition'] = div.find(class_="ik d-block w-100 ik interact").get("data-bs-content").split('\r\n')[2].split('>')[1]
    forecast_data['temperature_max'] = div.find(class_="ik max").get_text(strip=True)
    forecast_data['temperature_min'] = div.find(class_="ik min").get_text(strip=True)
    if div.find(class_="ik rainlevel-container") != None:
        if div.find(class_="ik rainlevel-container").a.span.get_text(strip=True) == ".":
            forecast_data['rain_mm'] = 1
        else:
            forecast_data['rain_mm'] = div.find(class_="ik rainlevel-container").a.span.get_text(strip=True).split(" ")[0]
    else:
        forecast_data['rain_mm'] = 0
    forecast_data_full.append(forecast_data)

forecast_daily_df = pd.DataFrame(forecast_data_full)

dictionary_path1 = 'file_path'
date_hour_format = converted_datetime.strftime("%Y-%m-%d_%H")
file_path1 = dictionary_path1 + date_hour_format
forecast_daily_df.to_csv(file_path1, index=False, header=False)

file_path2 = 'file_path'
forecast_daily_df.to_csv(file_path2, mode='a', index=False, header=False)



#Hourly_data:

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import datetime
import pytz
from bs4 import BeautifulSoup as bs
import requests

## download forecast data

url = "https://www.idokep.hu/elorejelzes/Budapest"
response = requests.get(url)
html = response.content
soup_forecast = bs(html, "lxml")

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)
i = 1
j = 1
forecast_data_full = []
for div in soup_forecast.find_all("div", class_="ik new-hourly-forecast-card"):
    forecast_data = {}
    forecast_data['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
    forecast_data['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
    forecast_data['actual_hour'] = converted_datetime.strftime("%H")
    forecast_data['version'] = "t-" + str(i)
    i = i + 1
    forecast_data['forecast_date'] = (converted_datetime + datetime.timedelta(hours=j)).strftime("%Y-%m-%d")
    j = j + 1
    forecast_data['forecast_hour'] = div.find(class_="ik new-hourly-forecast-hour").get_text(strip=True).split(":")[0]
    forecast_data['condition'] = div.a.get("data-bs-content")
    forecast_data['temperature'] = div.find(class_="tempValue").get_text(strip=True)
    forecast_data_full.append(forecast_data)

## download actual data
url = "https://www.idokep.hu/idojaras/Budapest"
response = requests.get(url)
html = response.content
soup_actual = bs(html, "lxml")

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)
actual_data = {}
actual_data['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
actual_data['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
actual_data['actual_hour'] = converted_datetime.strftime("%H")
actual_data['version'] = "t-0"
actual_data['forecast_date'] = converted_datetime.strftime("%Y-%m-%d")
actual_data['forecast_hour'] = converted_datetime.strftime("%H")
actual_data['condition'] = soup_actual.find(class_="ik current-weather").get_text(strip=True).lower()
actual_data['temperature'] = soup_actual.find(class_="ik current-temperature").get_text(strip=True).split("˚")[0]

forecast_data_full.append(actual_data)

forecast_df = pd.DataFrame(forecast_data_full)

dictionary_path1 = 'file_path'
date_hour_format = converted_datetime.strftime("%Y-%m-%d_%H")
file_path1 = dictionary_path1 + date_hour_format
forecast_df.to_csv(file_path1, index=False, header=False)

file_path2 = 'file_path'
forecast_df.to_csv(file_path2,mode='a',  index=False, header=False)

```
