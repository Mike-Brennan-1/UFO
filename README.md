# UFOs (Unidentified Frequency Outliers): Geospatial modelling and multivariate statistical analysis of reported UFO sightings

## Executive Summary

Using a publicly available dataset of over 80,000 reported sightings of unidentified flying objects (UFOs), spanning from 1906-2014, this project analysed geospatial patterns in reporting behaviour, specifically: whether proximity to an airport predicts the frequency of UFO sightings. Using SciPy’s cKDTree for geospatial nearest-neighbour lookup, each sighting’s nearest airport was determined using the Global Airports dataset from the World Bank. Euclidian distance between the sighting and the airport was calculated, and – after normalising by population density, using GeoTIFF files accessed from the World Pop Open Data Repository - the frequency of sightings reported in the United States was analysed versus distance from an airport. 

Histograms using Matplotlib and map visualisations built in both Folium and Power BI were created to demonstrate the project’s findings: that distance from an airport has a moderately negative correlation with frequency of reported UFO sightings, with sightings most frequently occurring within 100-150km of an airport.

## Data

The principal dataset used in this project was accessed from Kaggle.com (Kaggle, 2025). The data was originally "scraped, geolocated, and standardised" from the National UFO Reporting Centre, an online platform where users can submit reports of UFO sightings. CITATION

In addition, the Global Airports dataset, accessed via the World Bank was used to provide geospatial information of all the world's airports. (The World Bank, 2025). GeoTIFF datasets containing population density data for each square kilometre of the United States from 2000 to 2014 was accessed from World Pop, and processed using rasterio CITATION in a GeoPandas dataframe.

### Pre-processing

The UFO Sightings and Global Airports datasets were downloaded as .csv files and imported into a Pandas dataframe in Python.
In order to handle missing or invalid latitude and longitude values, these columns were converted to floats, with `errors = "coerce"`, meaning that any invalid data would be set to `NaN`. Any such rows were then dropped from the dataframe. Correct latitude and longitude columns were both crucial to the project; it was therefore important to ensure zero missing values, and interpolation the values was decided against, especially as the initial dataset contained only one invalid row.
```
#convert lang/long columns to numeric. Set errors to "coerce" and drop na to remove missing values

UFO_df["latitude"] = pd.to_numeric(UFO_df["latitude"], errors = "coerce")
UFO_df["longitude"] = pd.to_numeric(UFO_df["longitude"], errors = "coerce")

UFO_df.dropna(subset=["longitude", "latitude"], inplace = True)

#convert to datetime
UFO_df["datetime"] = UFO_df["datetime"].astype(str).str.replace("24:00", "00:00")
UFO_df["datetime"] = pd.to_datetime(UFO_df["datetime"])

UFO_df.head()
```
## Exploratory Data Analysis
By plotting a histogram of reported UFO sightings by year, three distinct epochs appeared evident: pre 1960s, where the line is almost flat on a linear graph, the latter part of the 20th century (1960s-1990s), and from the mid-1990s onwards.
![image](https://github.com/user-attachments/assets/60c81eab-ac96-4fbf-82eb-314bfc14be8e)




