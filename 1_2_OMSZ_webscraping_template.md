```python

#Forecast_data:

from selenium import webdriver
from selenium.webdriver import Chrome
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

service = Service(executable_path="file_path")
options = webdriver.ChromeOptions()
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
driver = webdriver.Chrome(service=service, options=options)
driver.get("https://www.met.hu/idojaras/elorejelzes/magyarorszagi_telepulesek/main.php?v=Budapest&c=tablazat&")
time.sleep(10)
page_source = driver.page_source
from bs4 import BeautifulSoup
soup_forecast = BeautifulSoup(page_source, 'html.parser')
driver.quit()

import numpy as np
import pandas as pd
import datetime
import pytz

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)

forecast_data_OMSZ_full = []
help1 = soup_forecast.find_all("tbody")[1]
help2 = help1.find_all("tr")
i = 1
j = 0
a = 0
for tr in help2:
    forecast_data_OMSZ = {}
    forecast_data_OMSZ['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
    forecast_data_OMSZ['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
    forecast_data_OMSZ['actual_hour'] = converted_datetime.strftime("%H")
    forecast_data_OMSZ['version'] = "t-" + str(i)
    i = i + 1
    forecast_data_OMSZ['forecast_date'] = (converted_datetime + datetime.timedelta(hours=a)).strftime("%Y-%m-%d")
    a = a + 6
    forecast_data_OMSZ['forecast_hour'] = help2[j].find(class_="ora rbg").get_text(strip=True).split(":")[0]
    if help2[j].find(class_="T X rbg0") != None:
        forecast_data_OMSZ['temp_max'] = help2[j].find(class_="T X rbg0").get_text(strip=True)
    elif help2[j].find(class_="T X rbg1") != None:
        forecast_data_OMSZ['temp_max'] = help2[j].find(class_="T X rbg1").get_text(strip=True)
    else:
        forecast_data_OMSZ['temp_max'] = ""
    if help2[j].find(class_="T N rbg1") != None:
        forecast_data_OMSZ['temp_min'] = help2[j].find(class_="T N rbg1").get_text(strip=True)
    elif help2[j].find(class_="T N rbg0") != None:
        forecast_data_OMSZ['temp_min'] = help2[j].find(class_="T N rbg0").get_text(strip=True)
    else:
        forecast_data_OMSZ['temp_min'] = ""
    if help2[j].find(class_="T rbg0") != None:
        forecast_data_OMSZ['temp'] = help2[j].find(class_="T rbg0").get_text(strip=True)
    elif help2[j].find(class_="T rbg1") != None:
        forecast_data_OMSZ['temp'] = help2[j].find(class_="T rbg1").get_text(strip=True)
    if help2[j].find(class_="R rbg0") != None:
        forecast_data_OMSZ['rain_mm'] = help2[j].find(class_="R rbg0").get_text(strip=True)
    elif help2[j].find(class_="R rbg1") != None:
        forecast_data_OMSZ['rain_mm'] = help2[j].find(class_="R rbg1").get_text(strip=True)
    if help2[j].find(class_="Wd rbg0") != None:
        forecast_data_OMSZ['wind'] = help2[j].find(class_="Wd rbg0").get_text(strip=True)
    elif help2[j].find(class_="Wd rbg1") != None:
        forecast_data_OMSZ['wind'] = help2[j].find(class_="Wd rbg1").get_text(strip=True)
    if help2[j].find(class_="Wf rbg0") != None:
        forecast_data_OMSZ['wind_kmh'] = help2[j].find(class_="Wf rbg0").get_text(strip=True)
        forecast_data_OMSZ['wind_max_kmh'] = help2[j].find_all(class_="Wf rbg0")[1].get_text(strip=True)
    elif help2[j].find(class_="Wf rbg1") != None:
        forecast_data_OMSZ['wind_kmh'] = help2[j].find(class_="Wf rbg1").get_text(strip=True)
        forecast_data_OMSZ['wind_max_kmh'] = help2[j].find_all(class_="Wf rbg1")[1].get_text(strip=True)
    if help2[j].find(class_="P rbg0") != None:
        forecast_data_OMSZ['pressure'] = help2[j].find(class_="P rbg0").get_text(strip=True)
    elif help2[j].find(class_="P rbg1") != None:
        forecast_data_OMSZ['pressure'] = help2[j].find(class_="P rbg1").get_text(strip=True)
    j = j + 1
    forecast_data_OMSZ_full.append(forecast_data_OMSZ)

forecast_data_OMSZ_full_df = pd.DataFrame(forecast_data_OMSZ_full)

dictionary_path1 = 'file_path'
date_hour_format = converted_datetime.strftime("%Y-%m-%d_%H")
file_path1 = dictionary_path1 + date_hour_format
forecast_data_OMSZ_full_df.to_csv(file_path1, index=False, header=False)

file_path2 = 'file_path'
forecast_data_OMSZ_full_df.to_csv(file_path2, mode='a', index=False, header=False)


#Actual_data:

import numpy as np
import pandas as pd
import datetime
import pytz
from bs4 import BeautifulSoup as bs
import requests

## download forecast data

url = "https://www.met.hu/idojaras/aktualis_idojaras/mert_adatok/main.php?v=Budapest&c=tablazat&"
response = requests.get(url)
html = response.content
soup_actual = bs(html, "lxml")

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)

actual_data_OMSZ_full = []
help1 = soup_actual.find_all("tbody")[0]
help2 = help1.find_all("tr")
i = 1
j = 0
a = 0
for j in range(0,24):
    actual_data_OMSZ = {}
    actual_data_OMSZ['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
    actual_data_OMSZ['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
    actual_data_OMSZ['actual_hour'] = converted_datetime.strftime("%H")
    actual_data_OMSZ['version'] = "t-0"
    i = i + 1
    actual_data_OMSZ['forecast_date'] = converted_datetime.strftime("%Y-%m-%d")
    a = a + 6
    if help2[j].find(class_="rbg0") != None:
        actual_data_OMSZ['forecast_hour'] = help2[j].find(class_="rbg0").get_text(strip=True).split(' ')[2]
    else:
        actual_data_OMSZ['forecast_hour'] = help2[j].find(class_="rbg1").get_text(strip=True).split(' ')[2]
    if help2[j].find(class_="T rbg0") != None:
        actual_data_OMSZ['temp_max'] = help2[j].find(class_="T rbg0").get_text(strip=True)
    elif help2[j].find(class_="T rbg1") != None:
        actual_data_OMSZ['temp_max'] = help2[j].find(class_="T rbg1").get_text(strip=True)
    if help2[j].find(class_="T rbg0") != None:
        actual_data_OMSZ['temp_min'] = help2[j].find(class_="T rbg0").get_text(strip=True)
    elif help2[j].find(class_="T rbg1") != None:
        actual_data_OMSZ['temp_min'] = help2[j].find(class_="T rbg1").get_text(strip=True)
    if help2[j].find(class_="T rbg0") != None:
        actual_data_OMSZ['temp'] = help2[j].find(class_="T rbg0").get_text(strip=True)
    elif help2[j].find(class_="T rbg1") != None:
        actual_data_OMSZ['temp'] = help2[j].find(class_="T rbg1").get_text(strip=True)
    if help2[j].find(class_="R rbg0") != None:
        actual_data_OMSZ['rain_mm'] = help2[j].find(class_="R rbg0").get_text(strip=True)
    elif help2[j].find(class_="R rbg1") != None:
        actual_data_OMSZ['rain_mm'] = help2[j].find(class_="R rbg1").get_text(strip=True)
    if help2[j].find(class_="Wd rbg0") != None:
        actual_data_OMSZ['wind'] = help2[j].find(class_="Wd rbg0").get_text(strip=True)
    elif help2[j].find(class_="Wd rbg1") != None:
        actual_data_OMSZ['wind'] = help2[j].find(class_="Wd rbg1").get_text(strip=True)
    if help2[j].find(class_="Wf rbg0") != None:
        actual_data_OMSZ['wind_kmh'] = help2[j].find(class_="Wf rbg0").get_text(strip=True)
        actual_data_OMSZ['wind_max_kmh'] = help2[j].find_all(class_="Wf rbg0")[1].get_text(strip=True)
    elif help2[j].find(class_="Wf rbg1") != None:
        actual_data_OMSZ['wind_kmh'] = help2[j].find(class_="Wf rbg1").get_text(strip=True)
        actual_data_OMSZ['wind_max_kmh'] = help2[j].find_all(class_="Wf rbg1")[1].get_text(strip=True)
    if help2[j].find(class_="P rbg0") != None:
        actual_data_OMSZ['pressure'] = help2[j].find(class_="P rbg0").get_text(strip=True)
    elif help2[j].find(class_="P rbg1") != None:
        actual_data_OMSZ['pressure'] = help2[j].find(class_="P rbg1").get_text(strip=True)
    j = j + 1
    actual_data_OMSZ_full.append(actual_data_OMSZ)

actual_data_OMSZ_full_df = pd.DataFrame(actual_data_OMSZ_full)

dictionary_path3 = 'file_path'
date_hour_format = converted_datetime.strftime("%Y-%m-%d_%H")
file_path3 = dictionary_path3 + date_hour_format
actual_data_OMSZ_full_df.to_csv(file_path3, index=False, header=False)
file_path4 = 'file_path'
actual_data_OMSZ_full_df.to_csv(file_path4, mode='a', index=False, header=False)

```
