# UFOs (Unidentified Frequency Outliers): Geospatial modelling and multivariate statistical analysis of reported UFO sightings

## Executive Summary

Using a publicly available dataset of over 80,000 reported sightings of unidentified flying objects (UFOs), spanning from 1906-2014, this project analysed geospatial patterns in reporting behaviour, specifically: whether proximity to an airport predicts the frequency of UFO sightings. Using SciPy’s cKDTree for geospatial nearest-neighbour lookup, each sighting’s nearest airport was determined using the Global Airports dataset from the World Bank. Euclidian distance between the sighting and the airport was calculated, and – after normalising by population density, using GeoTIFF files accessed from the World Pop Open Data Repository - the frequency of sightings reported in the United States was analysed versus distance from an airport. 

Histograms using Matplotlib and map visualisations built in both Folium and Power BI were created to demonstrate the project’s findings: that distance from an airport has a moderately negative correlation with frequency of reported UFO sightings, with sightings most frequently occurring within 100-150km of an airport.
