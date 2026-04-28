# data loading
import pandas as pd

try:
    crop_df = pd.read_csv("data/crop_yield.csv")
    print("Dataset loaded successfully!")

except FileNotFoundError:
    print("Error: File not found.")

except pd.errors.EmptyDataError:
    print("Error: File is empty.")

except Exception as e:
    print(f"Unexpected error: {e}")



# print 5 rows
print(crop_df.head())

# print column names
print(crop_df.columns)

# print info
print(crop_df.info)

# summary statistics
print(crop_df.describe())

total_unique_values_of_column = crop_df['crop'].nunique()
print(f"Total unique values in 'crop' column: {total_unique_values_of_column}")

missing_values_count = crop_df.isnull().sum()
print("Missing values in each column:", missing_values_count)

missing_crops = crop_df['crop'].isnull().sum()
print("\nRows with missing 'crop' values:", missing_crops)

# remove rows with missing 'yield' values
crop_df_cleaned = crop_df[crop_df['yield'].notna()] # equallent to notnull()
print(f"Number of rows after removing missing 'yield' values: {len(crop_df_cleaned)}")

# remove missing values from a DataFrame.
crop_df_filtered = crop_df.dropna();

# import numpy for numerical operations
import numpy as np

# identify numeric columns
numeric_columns = crop_df.select_dtypes(include=[np.number]).columns
print("Numeric columns:", numeric_columns)

for col in numeric_columns:
    missing_count = crop_df[col].isnull().sum()

    if missing_count > 0:
        mean_value = crop_df[col].mean()
        crop_df[col] = crop_df[col].fillna(mean_value)
        print(f"Filled {missing_count} missing values in '{col}' with mean: {mean_value}")

    else:
        print(f"No missing values in '{col}' to fill.")


# calculate number of rows in dataset
num_rows = len(crop_df)
print(f"Number of rows before removing duplicates: {num_rows}")

# remove duplicates
crop_df = crop_df.drop_duplicates()
print(f"Number of rows after removing duplicates: {len(crop_df)}")

#calculate average yield
avg_yield = crop_df['yield'].mean()
print(f"Average yield: {avg_yield}")

# create a new column 'is_high_yield'
crop_df['is_high_yield'] = crop_df['yield'] >= avg_yield
print(crop_df.head())

# create binary column 'is_high_yield' => 0 or 1
crop_df['binary_is_high_yeild'] = (crop_df['yield'] >= avg_yield).astype(int)
print(crop_df.head())

# import soil data
try:
    soil_df = pd.read_csv("data/state_soil_data.csv")
    print("Soil dataset loaded successfully!")
except FileNotFoundError:
    print("Error: Soil data file not found.")
except Exception as e:
    print(f"Unexpected error loading soil data: {e}")

print(soil_df.head())

# calculate nutrient index
soil_df['nutrient_index'] = (soil_df['N'] + soil_df['P'] + soil_df['K']) / soil_df['pH']
print(soil_df.head())

# round pH values to 1 decimal place
rounded_ph = soil_df['pH'].round(1)
print(rounded_ph.head())

# get unique velues using set
unique_ph = set(rounded_ph)
print(f"Unique pH values (rounded to 1 decimal place): {unique_ph}")

# count frequency of each unique pH value
freq_dict = {val: (rounded_ph == val).sum() for val in unique_ph}

# get top 5 most frequent
top_5_ph = sorted(freq_dict.items(), key=lambda x: x[1], reverse=True)[:5]

print("Top 5 most frequent pH values (rounded to 1 decimal place):", top_5_ph)

# create a temp level column and fill it based on temperature as low, medium and high

temp_val = [] # create an empty list to store the level values

weather_df = pd.read_csv("data/state_weather_data_1997_2020.csv")

# loop through the temperature column and assign levels
for temp in weather_df['avg_temp_c']:
    if pd.notna(temp):
        if temp < 20:
            temp_val.append('low')
        elif 20 < temp < 30:
            temp_val.append('medium')
        else:
            temp_val.append('high')
    else:
        temp_val.append('unknown') # append unknown for missing values

# add the new column to the weather_df
weather_df['temperature_level'] = temp_val

print(weather_df.head())

# count the frequency of each temperature level
level_counts = {}

for level in temp_val:
    if level in level_counts:
        level_counts[level] += 1
    else:
        level_counts[level] = 1

print("Frequency of each temperature level:", level_counts)

# last 5 rows of weather data
print("Tail: ", weather_df.tail())



-------------------------------------------------------------------------------------

import numpy as np

from logger_config import setup_logger
logger = setup_logger("SmartAgri")

logger.info(f"Smart Agriculture System initialized")

class CropRecord:
    def __init__(self, date:str, soil_moisture:float, temperature:float, 
    fertilizer_usage:float, crop_yield:float):

        # validate soil_moisture
        if soil_moisture is None or (isinstance(soil_moisture, float) and np.isnan(soil_moisture)):
            raise ValueError("soil_moisture is missing or invalid")
        
        # validate temperature
        if temperature is None or (isinstance(temperature, float) and np.isnan(temperature)):
            raise ValueError("temperature is missing or invalid")

        self.date = date
        self._soil_moisture = soil_moisture
        self._temperature = temperature
        self._fertilizer_usage = fertilizer_usage if fertilizer_usage is not None else None
        self._crop_yield = crop_yield if crop_yield is not None else None

        # access protected attributes via properties
        @property
        def soil_moisture(self):
            return self.soil_moisture
        
        @property
        def temperature(self):
            return self.temperature
        
        @property
        def fertilizer_usage(self):
            return self.fertilizer_usage
        
        @property
        def crop_yield(self):
            return self.crop_yield

class Crop:
    def __init__(self, cropType: str):
        self.cropType = cropType
        self._records: list[CropRecord] = []

    @property
    def records(self):
        return self._records
    
    def add_record(self, record: CropRecord):
        if not isinstance(record, CropRecord):
            raise ValueError("record must be an instance of CropRecord")
        self._records.append(record)
        logger.info(f"Added record for crop {self.cropType} on date {record.date}")

class Farm:
    def __init__(self, farm_id:str, location:str):
        self._farm_id = farm_id
        self._location = location
        self._crops: dict[str, Crop] = {}


# logger_config file
import logging

def setup_logger(name: str):
    logger = logging.getLogger(name)

    if not logger.handlers:  # prevent duplicate logs
        logger.setLevel(logging.INFO)

        formatter = logging.Formatter(
            "%(asctime)s | %(levelname)-7s | %(name)s | %(message)s"
        )

        file_handler = logging.FileHandler("farm_platform.log", mode="w")
        file_handler.setFormatter(formatter)

        console_handler = logging.StreamHandler()
        console_handler.setFormatter(formatter)

        logger.addHandler(file_handler)
        logger.addHandler(console_handler)

    return logger