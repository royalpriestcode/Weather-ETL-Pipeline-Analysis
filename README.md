# Weather ETL Pipeline

## Project Overview

This project demonstrates a simple ETL (Extract, Transform, Load) pipeline built with Python.

The pipeline extracts current weather data for Lagos, Ibadan, and Abuja from the OpenWeather API, transforms the raw API response into a structured Pandas DataFrame, stores the processed data as a CSV file, and performs basic comparative analysis on the resulting dataset.

The project was designed to demonstrate how data can be collected from an external API, cleaned and structured programmatically, stored for reuse, and analysed using Python.

---

## Data Source

The data used in this project was obtained from the OpenWeather Current Weather API.

The API provides current weather information for different locations. For this project, weather data was collected for:

- Lagos
- Ibadan
- Abuja

The main variables collected were:

- City Name
- Temperature
- Humidity
- Weather Condition
- Wind Speed
- Date and Time

The OpenWeather endpoint used for the project was:

```python
url = "https://api.openweathermap.org/data/2.5/weather"
```

The API key was stored privately using Google Colab Secrets rather than being written directly inside the notebook.

```python
from google.colab import userdata

API_KEY = userdata.get("OPENWEATHER_API_KEY")
```

---

## ETL Process

### 1. Extract

The first stage of the pipeline involved extracting current weather data from the OpenWeather API.

The required Python libraries were imported before making the API request.

```python
import requests
import pandas as pd
from datetime import datetime
from google.colab import userdata
```

The API key was retrieved securely from Google Colab Secrets.

```python
API_KEY = userdata.get("OPENWEATHER_API_KEY")
```

The OpenWeather endpoint and the list of cities were then defined.

```python
url = "https://api.openweathermap.org/data/2.5/weather"

cities = ["Lagos", "Ibadan", "Abuja"]
```

An empty list was created to store the weather record returned for each city.

```python
weather_records = []
```

A loop was then used to send an HTTP GET request for every city in the list.

```python
for city in cities:

    params = {
        "q": city,
        "appid": API_KEY,
        "units": "metric"
    }

    response = requests.get(url, params=params)

    response.raise_for_status()

    data = response.json()
```

The `requests.get()` method sent the request to the OpenWeather API.

The parameters supplied included:

- `q` for the city name
- `appid` for authentication using the API key
- `units="metric"` so that temperature values were returned in Celsius

The `raise_for_status()` method was used to stop the program if the API request returned an unsuccessful HTTP status code.

The API response was then converted from JSON into a Python object using:

```python
data = response.json()
```

---

### 2. Transform

After extracting the raw API response, only the weather variables required for the project were selected.

A dictionary was created for each city containing the relevant fields.

```python
record = {
    "City": data["name"],
    "Temperature_C": data["main"]["temp"],
    "Humidity_Pct": data["main"]["humidity"],
    "Weather_Condition": data["weather"][0]["main"],
    "Wind_Speed_mps": data["wind"]["speed"],
    "Date_Time": datetime.fromtimestamp(data["dt"])
}
```

Each dictionary was added to the `weather_records` list.

```python
weather_records.append(record)
```

After all three cities had been processed, the list of dictionaries was converted into a Pandas DataFrame.

```python
weather_df = pd.DataFrame(weather_records)
```

The dataset was inspected to understand its structure, data types, and completeness.

```python
weather_df.info()
```

Missing values were checked using:

```python
weather_df.isnull().sum()
```

Duplicate records were also checked.

```python
weather_df.duplicated().sum()
```

The `Date_Time` field was converted into a Pandas datetime format.

```python
weather_df["Date_Time"] = pd.to_datetime(
    weather_df["Date_Time"]
)
```

Separate date and time columns were created from the datetime field.

```python
weather_df["Date"] = weather_df["Date_Time"].dt.date
weather_df["Time"] = weather_df["Date_Time"].dt.time
```

The dataset was then sorted by temperature in descending order.

```python
weather_df = weather_df.sort_values(
    by="Temperature_C",
    ascending=False
)
```

Finally, the DataFrame index was reset after sorting.

```python
weather_df = weather_df.reset_index(drop=True)
```

At the end of the transformation stage, the raw API responses had been converted into a clean and structured dataset suitable for storage and analysis.

---

### 3. Load

After the transformation stage, the processed DataFrame was saved as a CSV file.

```python
weather_df.to_csv(
    "weather_data.csv",
    index=False
)
```

The `index=False` argument prevented the Pandas row index from being written as an additional column in the CSV file.

The stored CSV file was then loaded back into Pandas to confirm that it had been created successfully.

```python
saved_weather = pd.read_csv("weather_data.csv")

saved_weather
```

This confirmed that the processed weather dataset had been successfully stored as:

```text
weather_data.csv
```

---

## Tools Used

The following tools and technologies were used in the project:

- **Python** — used to build the ETL pipeline and perform the analysis
- **Requests** — used to make HTTP requests to the OpenWeather API
- **Pandas** — used for transforming, cleaning, structuring, and analysing the data
- **OpenWeather API** — used as the external source of current weather data
- **Google Colab** — used as the development environment
- **Google Colab Secrets** — used to store the API key securely
- **CSV** — used as the final storage format for the processed dataset
- **GitHub** — used for version control, documentation, and project presentation

---

## Steps Taken

The project followed the workflow below:

1. Created an OpenWeather account and obtained an API key.
2. Tested the API connection using one city.
3. Confirmed successful authentication using an HTTP `200` response.
4. Defined Lagos, Ibadan, and Abuja as the cities for the project.
5. Used a Python loop to send API requests for each city.
6. Converted each JSON response into a Python object.
7. Extracted only the required weather variables.
8. Stored each city's weather record as a dictionary.
9. Added the dictionaries into a list.
10. Converted the list of dictionaries into a Pandas DataFrame.
11. Inspected the DataFrame structure and data types.
12. Checked for missing values.
13. Checked for duplicate records.
14. Converted the datetime field into a usable Pandas datetime format.
15. Created separate date and time columns.
16. Sorted the dataset by temperature.
17. Saved the transformed dataset as a CSV file.
18. Loaded the CSV file back into Pandas to verify the output.
19. Performed basic comparative analysis across the three cities.

---

## Data Analysis

### Temperature Comparison

The temperature recorded for each city was displayed for comparison.

```python
weather_df[["City", "Temperature_C"]]
```

The city with the highest temperature was identified using `idxmax()`.

```python
hottest_city = weather_df.loc[
    weather_df["Temperature_C"].idxmax(),
    ["City", "Temperature_C"]
]

hottest_city
```

The city with the lowest temperature was identified using `idxmin()`.

```python
coolest_city = weather_df.loc[
    weather_df["Temperature_C"].idxmin(),
    ["City", "Temperature_C"]
]

coolest_city
```

The average temperature across the three cities was calculated using the `mean()` method.

```python
average_temperature = weather_df["Temperature_C"].mean()

round(average_temperature, 2)
```

### Humidity Comparison

The city with the highest humidity was identified using:

```python
highest_humidity = weather_df.loc[
    weather_df["Humidity_Pct"].idxmax(),
    ["City", "Humidity_Pct"]
]

highest_humidity
```

### Weather Condition Comparison

The weather condition for each city was displayed.

```python
weather_df[["City", "Weather_Condition"]]
```

The frequency of each weather condition was also calculated.

```python
weather_df["Weather_Condition"].value_counts()
```

### Wind Speed Comparison

The city with the highest wind speed was identified using:

```python
highest_wind = weather_df.loc[
    weather_df["Wind_Speed_mps"].idxmax(),
    ["City", "Wind_Speed_mps"]
]

highest_wind
```

### Descriptive Statistics

Basic descriptive statistics were generated for the numerical variables.

```python
weather_df.describe()
```

This produced summary measures such as count, mean, standard deviation, minimum, maximum, and quartiles.

---

## Key Findings

Based on the weather data collected at the time of extraction:

- **Abuja** recorded the highest temperature at **28.08°C**.
- **Ibadan** recorded the lowest temperature at **26.68°C**.
- The average temperature across the three cities was approximately **27.51°C**.
- **Ibadan** recorded the highest humidity level at **80%**.
- **Ibadan** and **Abuja** both recorded **Rain** as their weather condition.
- **Lagos** recorded **Clouds** as its weather condition.
- **Lagos** recorded the highest wind speed at **3.57 m/s**.

These findings represent weather conditions at the specific time the API requests were made and should not be interpreted as long-term climatic characteristics of the cities.

---

## Conclusion

This project demonstrates a complete ETL workflow using Python, from extracting current weather data through an external API to transforming, storing, and analysing the resulting dataset.

It also demonstrates how APIs, Python data structures, Pandas, secure credential handling, and structured data processing can be combined to create a simple and reusable data pipeline.
