# UFOs (Unidentified Frequency Outliers): Geospatial modelling and multivariate statistical analysis of reported UFO sightings

## Executive Summary

Using a publicly available dataset of over 80,000 reported sightings of unidentified flying objects (UFOs), spanning from 1906-2014, this project analysed geospatial patterns in reporting behaviour, specifically: whether proximity to an airport predicts the frequency of UFO sightings. Using SciPy’s cKDTree for geospatial nearest-neighbour lookup, each sighting’s nearest airport was determined using the Global Airports dataset from the World Bank. Euclidian distance between the sighting and the airport was calculated, and – after normalising by population density, using GeoTIFF files accessed from the World Pop Open Data Repository - the frequency of sightings reported in the United States was analysed versus distance from an airport. 

Histograms using Matplotlib and map visualisations built in both Folium and Power BI were created to demonstrate the project’s findings: that distance from an airport has a moderately negative correlation with frequency of reported UFO sightings, with sightings most frequently occurring within 100-150km of an airport. A linear regression model suggested a statistically significant (p<0.001) finding: that for every 10km distance form an airport, the number of sightings drops by approximately 7.6.


## Data

The principal dataset used in this project was accessed from Kaggle.com (Kaggle, 2025). The data was originally "scraped, geolocated, and standardised" from the National UFO Reporting Centre, an online platform where users submit reports of UFO sightings. (NUFORC, 2025)

In addition, the Global Airports dataset, accessed via the World Bank was used to provide geospatial information of all the world's airports. (The World Bank, 2025). GeoTIFF datasets containing population density data for each square kilometre of the United States from 2000 to 2014 was accessed from World Pop (WorldPop, 2020).

### Pre-processing

The UFO Sightings and Global Airports datasets were downloaded as .csv files and imported into a Pandas dataframe in Python.
To handle missing or invalid geospatial values, these columns were converted to floats, with `errors = "coerce"`, meaning that any invalid data would be set to `NaN`. Any such rows were then dropped from the dataframe. Correct latitude and longitude columns were both crucial to the project; it was therefore important to ensure zero missing values, and interpolation the values was decided against.

```Python
#convert lang/long columns to numeric. Set errors to "coerce" and drop na to remove missing values

UFO_df["latitude"] = pd.to_numeric(UFO_df["latitude"], errors = "coerce")
UFO_df["longitude"] = pd.to_numeric(UFO_df["longitude"], errors = "coerce")

UFO_df.dropna(subset=["longitude", "latitude"], inplace = True)

#convert to datetime
UFO_df["datetime"] = UFO_df["datetime"].astype(str).str.replace("24:00", "00:00")
UFO_df["datetime"] = pd.to_datetime(UFO_df["datetime"])

UFO_df.head()
```
> *Figure 16 Code snippet showing removal of invalid latitude / longitude rows*

## Exploratory Data Analysis
By plotting a histogram of reported UFO sightings by year, three distinct epochs appeared evident: pre 1960s, where the line is almost flat on a linear graph, the latter part of the 20th century (1960s-1990s), and from the mid-1990s onwards.

![image](https://github.com/user-attachments/assets/60c81eab-ac96-4fbf-82eb-314bfc14be8e)
> *Figure 17 Frequency of reported UFO sightings by year*

A similar plot, showing the frequency of sightings per month, shows that reported sightings tend to peak in the summer, and decrease in the winter months.

![image](https://github.com/user-attachments/assets/a346a328-eba0-4a7d-b462-5c24f8b9f9eb)
> *Figure 18 Frequency of reported UFO sightings by month*

## Visualising UFO reports in Power BI
In order to allow those engaging with this report the opportunity to explore and interact with the dataset, an interactive Power BI map visualisation was created, as Figures 20 & 21 show. This visual allows users to filter reports by date, nearest airport and distance from an airport. The tooltips provide a short description of the sighting, the date and time, and information about the nearest airport.

```Python
#Handle HTML encoding issues and export as csv for use in PBI map visual

import html

UFO_df["comments"] = UFO_df["comments"].apply(lambda x: html.unescape(str(x)) if pd.notnull(x) else x)

airport_cols = airports_df[["Orig", "Name"]]
UFO_df = UFO_df.merge(airport_cols, left_on='nearest_airport', right_on='Orig', how='left')
UFO_df["Airport display name"] = UFO_df["Name"] + " (" + UFO_df['nearest_airport'] + ")"


UFO_df.drop(columns=["city", "state", "shape", "duration (seconds)", "duration (hours/min)", "date posted", "nearest_airport", "Orig", "Name"], inplace = True)
UFO_df.to_csv("UFO_pbi_data.csv", index = False)
```
> *Figure 19 Code snippet showing the export of the dataset for use in a Power BI visual. The initial dataset contained HTML elements in the “comments” column. These were converted to plaintext for better interpretability.*

![image](https://github.com/user-attachments/assets/6eaf02d6-d938-4a3b-ac8d-51c2bd5902e5)
> *Figure 20 Screenshot of the Power BI map visualisation, filtering sightings by date.*

![image](https://github.com/user-attachments/assets/d08eef9c-0b08-4bd4-803d-db0dd9954b33)
> *Figure 21  Snapshot of Power BI visual, displaying sightings near London Heathrow airport from 1960 onwards. The tooltips show some basic information about the report, the location, and the nearest airport.*

## The Airport Hypothesis

| Hypothesis | Test |
| --------------- | --------------- |
| **H₀**: Distance from an airport has no effect on frequency of UFO sightings | No statistically significant correlation between distance from airport and frequency of sightings |
| **H₁**: The closer you get to an airport, the more likely you are to report a UFO | Distance from an airport and frequency of UFO reports negatively correlate, with *p* < 0.05 |

> *Table 4 Hypothesis statement*

One common explanation for alleged UFO sightings is that people can mistake conventional aircraft flying overhead for alien spacecraft. To test the hypothesis, the geospatial data associated with each sighting was used to calculate the distance to the nearest airport; airport locations were determined using the Global Airports dataset.

### Geospatial modelling: nearest neighbour lookup and distance calculation
```Python
from scipy.spatial import cKDTree

UFO_coordinates = UFO_df[["latitude", "longitude"]]
Airport_coordinates = airports_df[["Latitude", "Longitude"]]

airport_tree = cKDTree(Airport_coordinates)
distances, indices = airport_tree.query(UFO_coordinates)

UFO_df["nearest_airport"] = airports_df.iloc[indices]["Orig"].values
UFO_df["distance_to_airport_km"] = distances * 111 #approximate conversion from degress to kilometres

UFO_df.head()
```
> *Figure 22 To calculate the approximate distance in kilometres, Scipy's `cKDTree` class was used: an algorithm that permits efficient nearest-neighbour lookups for a k-dimensional array.*

For each row in the UFO dataframe, the airport code of the nearest airport was assigned to column `nearest_airport`. The distance between the two pairs of coordinates was then converted to kilometres by multiplying by 111. It should be noted that this conversion is approximate, as it assumes Euclidian space. A more accurate conversion could be achieved by using the haversine formula (Dauni, 2019), which accounts for the curvature of the Earth. For the proposes of calculating relative distance, however, an approximate value was deemed acceptable. If - in future iterations of the project - it were necessary to calculate precise flight paths, for example, then more accurate calculations would be appropriate.

![image](https://github.com/user-attachments/assets/7ff61c10-fbc5-4ef4-840b-b554ac3bbf02)
> *Figure 23 Number of UFO sightings vs distance to an airport (unfiltered). The distribution is so far skewed to the left, that the scale of the graph makes it difficult to interpret.*

![image](https://github.com/user-attachments/assets/a11c8b1a-4811-4617-bed4-936adc75e628)
> *Figure 24 By looking at distances of 300 kilometres or less, a clearer narrative emerges: a strong negative correlation between distance to an airport and frequency of sightings, with a drop towards the left of the distribution.*

### Controlling for population density
Figure 13 appears to prove a strong negative correlation between frequency of UFO sightings and distance to the nearest airport. There is a drop towards the left of the distribution, meaning fewer sightings were reported by people in extreme proximity to - or actually at - an airport, where aircraft would be flying lower, and people may be less likely to mistake them for a UFO. Prima facie, the graph appears to make a very strong case for the hypothesis that proximity to an airport predicts UFO sightings. Having said this, further analysis is required to control for population density. 
Airports are often located close to major urban centres, where there are more people. It could be reasonably assumed that the reported frequency of any type of human-observed phenomenon, terrestrial or otherwise, might increase the closer you get to an airport. The more people there are in any given location, the more people can submit reports.
```Python
years = range(2000, 2015)
UFO_by_year = {}

for y in years:
    UFO_by_year[y] = UFO_df[(UFO_df["datetime"].dt.year == y) & (UFO_df["country"] == "us")]
    print(f"year: {y} - {len(UFO_by_year[y])} rows")
```
```Text
year: 2000 - 2184 rows
year: 2001 - 2445 rows
year: 2002 - 2439 rows
year: 2003 - 2958 rows
year: 2004 - 3244 rows
year: 2005 - 3228 rows
year: 2006 - 2884 rows
year: 2007 - 3470 rows
year: 2008 - 4017 rows
year: 2009 - 3677 rows
year: 2010 - 3548 rows
year: 2011 - 4379 rows
year: 2012 - 6320 rows
year: 2013 - 6056 rows
year: 2014 - 1964 rows
```
> *Figure 25 After ensuring that the filtered dataframes, with year-specific reports from the USA, contained a sufficient number of rows for the analysis, the population density relating to the year and to the square kilometre of each sighting was added to a dictionary of Geopandas dataframes.*

At one-kilometre resolution, World Pop’s population density datasets are broken down by country and by year. Given that accessing hundreds of gigabytes’ worth of files would be impractical, and that the overwhelming majority of reports in the UFO dataset are located in the USA, it was decided to focus on reports from the United States from 2000 (the earliest year for which population density data is available) to 2014 (the latest year in the UFO dataset). 

```Python
UFO_gdfs = {}

for year, tif_path in tif_paths.items():
    gdf = gpd.GeoDataFrame(
    UFO_by_year[year],
    geometry=gpd.points_from_xy(UFO_by_year[year].longitude, UFO_by_year[year].latitude),
    crs="EPSG:4326" #specify the coordinate reference system used in the .tif
    )

    with rasterio.open(tif_path) as src:
        coords = [(pt.x, pt.y) for pt in gdf.geometry]
        values = [val[0] if val else None for val in src.sample(coords)]

    gdf["pop_density"] = values
    gdf = gdf.dropna(subset=["pop_density"])
    gdf["adjusted_sighting"] = 1 / gdf["pop_density"]
    UFO_gdfs[year] = gdf

UFO_gdfs[2009].head()
```
> *Figure 26  The column `”adjusted_sighting”` was calculated by taking the inverse of the population density*

$$
\text{Adjusted sighting} = \frac{1}{\text{population density}}
$$

Given that each row of the UFO dataset represents one sighting, and the purpose of normalising by population density is to isolate the variable of proximity to an airport without regard to population, taking the inverse of the population density both penalises reports from highly populated areas and promotes reports from sparsely populated areas. Crucially, all sightings were weighted to precisely the same degree with respect to population.

For example:

| Number of Reports | Population Density (per sq. km) | Adjusted Sighting|
|-------------------|-------------------------------|--------------------|
| 1                 | 1000                          | 1 / 1000 = 0.001   |
| 1                 | 500                           | 1 / 500 = 0.002    |

> *Table 5 A simplified example of how taking the inverse of population density normalises each reported sighting. A sighting reported in an area that is half as densely populated as another area, counts twice as much, and vice versa.*

It is important to note that this does not provide a true per capita metric. To do so, the dataset would need to be grouped into square-kilometre bins and the total sum of sightings divided by the population per bin. Given that the model is focused on distance from an airport, and is agnostic of actual location, the adjusted sighting column was deemed a valid proxy for sightings weighted by population density, which could be aggregated in the next step.

## Results
Frequency of reported UFO sightings, weighted by population, was plotted against distance to the nearest airport for the years 2000-2014. (Figure 27)
![image](https://github.com/user-attachments/assets/92637f6a-8137-40ac-b06f-09d53ae17b0c)
> *Figure 27 Histogram plots for the years 2000-2014 showing the frequency of reported UFO sightings (weighted to normalise by population density) in the US versus distance from an airport, plotted on a logarithmic scale due to the wide range of frequencies*

Despite some outliers and anomalies, for example, unusually high frequencies towards the middle and right of the distribution in 2001, 2004, 2009 and 2010, each year appears to mirror the overall pattern shown in the aggregate graph (Figure 28): a moderate negative correlation between distance from an airport and frequency of reported UFO sightings, with the distribution tending to peak at a threshold of 100-150 km and slightly dropping in the immediate vicinity of an airport.
![image](https://github.com/user-attachments/assets/d4b86375-5ae9-423a-ab64-e885d5913853)
> *Figure 28 Histogram of all reported UFO sightings (normalised by population density) for the years 2000-2014 in the US, plotted on a logarithmic scale, versus distance to an airport*

Figure 29 shows a heatmap of the reported sightings, weighted by population density from 2000-2014.

![image](https://github.com/user-attachments/assets/fcccdd9a-bdc7-49fb-b0c1-18c12c641a5d)
> *Figure 29 Heatmap of reported UFO sightings 2000-2014, weighted by population density.*

By way of comparison, Figure 30 , taken from an article published by MIT, shows volume of air traffic over the United States in 2004. (Mozdzanowska & Hansman, 2007)

![image](https://github.com/user-attachments/assets/244d6f18-ad97-4f0b-a4e8-34eedc34c9bb)
> *Figure 30 Volume of air traffic over the contiguous United States in 2004 (Mozdzanowska & Hansman, 2007)*

When compared to Figure 31, which shows reported sightings specifically from 2004, some similarities can be observed. Namely: the bulk of weighted sightings and of air traffic is in the Eastern half of the country. Much of the Mountain and Midwest states show low levels of both UFO reports and air traffic. Various hubs of activity can be observed on the West Coast and the Midwest: Los Angeles, San Jose, Seattle and Denver for instance.

![image](https://github.com/user-attachments/assets/a50ebe8f-7e3c-46a2-a275-415a9fcc0a92)
> *Figure 31  Heatmap of reported UFO sightings in 2004, weighted by population density.*

> [Click here to view an interactive version of this map](https://mike-brennan-1.github.io/UFO/docs/heatmap_2004.html)

### Statistical validation
Sightings were split into 10km bins according to `distance_from_airport_km`. A linear regression model was trained on the count of sightings per bin and the mean of the inverse population density (`sighting_adjusted`) as its features. The model confirmed a statistically significant negative correlation between distance from an airport and number of sightings. Based on the model output (Figure 32), the null hypothesis was rejected in favour of the alternative hypothesis (Table 4): distance from an airport negatively predicts reported UFO sightings.

```Text
OLS Regression Results                            
==============================================================================
Dep. Variable:         sighting_count   R-squared:                       0.335
Model:                            OLS   Adj. R-squared:                  0.303
Method:                 Least Squares   F-statistic:                     10.56
Date:                Mon, 05 May 2025   Prob (F-statistic):           0.000192
Time:                        09:09:53   Log-Likelihood:                -404.74
No. Observations:                  45   AIC:                             815.5
Df Residuals:                      42   BIC:                             820.9
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
============================================================================================
                               coef    std err          t      P>|t|      [0.025      0.975]
--------------------------------------------------------------------------------------------
const                     3167.4350    528.221      5.996      0.000    2101.441    4233.429
bin_midpoint                -7.6061      1.713     -4.441      0.000     -11.063      -4.149
mean_inverse_pop_density -1186.5798   1060.338     -1.119      0.269   -3326.429     953.270
==============================================================================
Omnibus:                       42.362   Durbin-Watson:                   0.376
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              141.907
Skew:                           2.467   Prob(JB):                     1.53e-31
Kurtosis:                      10.165   Cond. No.                     1.07e+03
==============================================================================
```

## Future iterations
Whereas the visualisations and linear regression model demonstrate a negative correlation between frequency of UFO sightings and distance to an airport, outliers remain in the data, and for some years, the corelation is more moderate.

To investigate the effect further, more precise distance calculation, using the haversine formula, combined with data on flight paths or airport capacity would permit greater isolation of the variable under scrutiny. 

As the model’s R-squared suggests, airport proximity accounts for 34% of the variance. It is likely that airport proximity is only one variable in a multivariate model that can help predict the location and frequency of reported UFO sightings. Future iterations of this project might include examining meteorological data to check visibility or the presence of unusual weather phenomena, data on home PC or smartphone ownership and its relationship to reporting behaviour over time, or events in popular culture – the release of a UFO-themed blockbuster, for example – and whether they coincide with spikes in reporting frequency.








