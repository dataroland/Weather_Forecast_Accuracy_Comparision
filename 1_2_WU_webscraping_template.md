```python

#Forecast_data:

import numpy as np
import pandas as pd
import datetime
import pytz
import time
import re
import json

from selenium import webdriver
from selenium.webdriver import Chrome
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

service = Service(executable_path="/home/dataroland/data_source_course/OMSZ/chromedriver")
options = webdriver.ChromeOptions()
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
driver = webdriver.Chrome(service=service, options=options)
driver.get("https://www.wunderground.com/forecast/hu/budapest/LHBP")
time.sleep(10)
page_source = driver.page_source
from bs4 import BeautifulSoup
IBM = BeautifulSoup(page_source, 'html.parser')
driver.quit()

temp = IBM.find_all("script", id="app-root-state")[0]
temp1 = temp.string
temp1 = temp1.replace("&q;", '"')
temp1 = temp1.strip().rstrip(';')
data_dict = json.loads(temp1)

temp = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['temperature']
timeLocal = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['validTimeLocal']
precipChance = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['precipChance']
precipType = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['precipType']
pressure = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['pressureMeanSeaLevel']
precip1Hour = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['qpf']
snow1Hour = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['qpfSnow']
humidity = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['relativeHumidity']
uvDescription = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['uvDescription']
windDir = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['windDirectionCardinal']
windGust = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['windGust']
wind_speed_mph = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['windSpeed']
condition = data_dict['wu-next-state-key']['b14694867cbf9436d6fdddc55620fae1']['value']['wxPhraseLong']

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)

forecast_data_IBM_full = []
i = 0
j = 1
for i in range(0,360):
    forecast_data_IBM = {}
    forecast_data_IBM['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
    forecast_data_IBM['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
    forecast_data_IBM['actual_hour'] = converted_datetime.strftime("%H")
    forecast_data_IBM['version'] = "t-" + str(j)
    j = j + 1
    forecast_data_IBM['forecast_date'] = timeLocal[i][0:10]
    forecast_data_IBM['forecast_hour'] = timeLocal[i][11:13]
    forecast_data_IBM['temp'] = round((temp[i] - 32) * 5/9)
    forecast_data_IBM['precip1Hour'] = precip1Hour[i]*25.4
    forecast_data_IBM['snow1Hour'] = snow1Hour[i]*25.4
    forecast_data_IBM['precipChance'] = precipChance[i]
    forecast_data_IBM['precipType'] = precipType[i]
    forecast_data_IBM['precip24Hour'] = ""
    forecast_data_IBM['snow24Hour'] = ""
    forecast_data_IBM['pressure'] = round(pressure[i]*33.863889532610884)
    forecast_data_IBM['humidity'] = humidity[i]
    forecast_data_IBM['uvDescription'] = uvDescription[i]
    forecast_data_IBM['windDir'] = windDir[i]
    forecast_data_IBM['windGust'] = windGust[i]
    forecast_data_IBM['wind_speed_mph'] = wind_speed_mph[i]
    forecast_data_IBM['condition'] = condition[i]
    i = i + 1
    forecast_data_IBM_full.append(forecast_data_IBM)

forecast_data_IBM_full_df = pd.DataFrame(forecast_data_IBM_full)

dictionary_path1 = '/home/dataroland/data_source_course/IBM_weather/IBM_forecast/'
date_hour_format = converted_datetime.strftime("%Y-%m-%d_%H")
file_path1 = dictionary_path1 + date_hour_format
forecast_data_IBM_full_df.to_csv(file_path1, index=False, header=False)

file_path2 = '/home/dataroland/data_source_course/IBM_weather/all_data/IBM_all_data.csv'
forecast_data_IBM_full_df.to_csv(file_path2, mode='a', index=False, header=False)



#Actual_data:

import numpy as np
import pandas as pd
import datetime
import pytz
import time
import re
import json

from selenium import webdriver
from selenium.webdriver import Chrome
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

service = Service(executable_path="/home/dataroland/data_source_course/OMSZ/chromedriver")
options = webdriver.ChromeOptions()
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
driver = webdriver.Chrome(service=service, options=options)
driver.get("https://www.wunderground.com/forecast/hu/budapest/LHBP")
time.sleep(10)
page_source = driver.page_source
from bs4 import BeautifulSoup
IBM = BeautifulSoup(page_source, 'html.parser')
driver.quit()

temp = IBM.find_all("script", id="app-root-state")[0]
temp1 = temp.string
temp1 = temp1.replace("&q;", '"')
temp1 = temp1.strip().rstrip(';')
data_dict = json.loads(temp1)

temp = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['temperature']
timeLocal = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['validTimeLocal']
precip24Hour = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['precip24Hour']
snow24Hour = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['snow24Hour']
pressure = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['pressureMeanSeaLevel']
humidity = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['relativeHumidity']
uvDescription = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['uvDescription']
windDir = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['windDirectionCardinal']
windGust = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['windGust']
wind_speed_mph = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['windSpeed']
condition = data_dict['wu-next-state-key']['a36951cdb6ab7935d8f6176de45bdb54']['value']['wxPhraseLong']

utc_datetime = datetime.datetime.now(pytz.utc)
budapest_timezone = pytz.timezone('Europe/Budapest')
converted_datetime = utc_datetime.astimezone(budapest_timezone)

actual_data_IBM_full = []
i = 0
for i in range(0,24):
    actual_data_IBM = {}
    actual_data_IBM['actual_datetime'] = converted_datetime.strftime("%Y-%m-%d %H:%M:%S")
    actual_data_IBM['actual_date'] = converted_datetime.strftime("%Y-%m-%d")
    actual_data_IBM['actual_hour'] = converted_datetime.strftime("%H")
    actual_data_IBM['version'] = "t-0"
    actual_data_IBM['forecast_date'] = timeLocal[i][0:10]
    actual_data_IBM['forecast_hour'] = timeLocal[i][11:13]
    actual_data_IBM['temp'] = round((temp[i] - 32) * 5/9)
    actual_data_IBM['precip1Hour'] = ""
    actual_data_IBM['snow1Hour'] = ""
    actual_data_IBM['precipChance'] = "100"
    actual_data_IBM['precipType'] = ""
    actual_data_IBM['precip24Hour'] = precip24Hour[i]*25.4
    actual_data_IBM['snow24Hour'] = snow24Hour[i]*25.4
    actual_data_IBM['pressure'] = round(pressure[i]*1)
    actual_data_IBM['humidity'] = humidity[i]
    actual_data_IBM['uvDescription'] = uvDescription[i]
    actual_data_IBM['windDir'] = windDir[i]
    actual_data_IBM['windGust'] = windGust[i]
    actual_data_IBM['wind_speed_mph'] = wind_speed_mph[i]
    actual_data_IBM['condition'] = condition[i]
    i = i + 1
    actual_data_IBM_full.append(actual_data_IBM)

actual_data_IBM_full_df = pd.DataFrame(actual_data_IBM_full)

dictionary_path1 = '/home/dataroland/data_source_course/IBM_weather/IBM_actual/'
date_hour_format = converted_datetime.strftime("%Y-%m-%d_%H")
file_path1 = dictionary_path1 + date_hour_format
actual_data_IBM_full_df.to_csv(file_path1, index=False, header=False)

file_path2 = '/home/dataroland/data_source_course/IBM_weather/all_data/IBM_all_data.csv'
actual_data_IBM_full_df.to_csv(file_path2, mode='a', index=False, header=False)

```
