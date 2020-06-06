# Water Storage Retrieval

Semi-supervised water storage retrieval using Sentinel-2 and *in situ* bathymetric data.

The pipeline is divided into three phases:

## Phase I: Automatic Water Classification

Water is automatically classificated in an object-based approach using Simple Non-Iterative Clustering (SNIC) and k-means.

First, water is automatically classified in `phase-I/01-automatic-water-classification.ipynb`. The Sentinel-2 image is preprocessed (median filtering is applied) and segmented using SNIC algorithm in Google Earth Engine. As inputs for the segmentation, the 10 m bands + the NDWI were used.

## Phase II: Satellite Derieved Bathymetry

Water depth is estimated through 3 different regression models: Linear Regression, Random Forest and Gradient Boosting.

## Phase III: Water Storage Retrieval

Total water storage volume is retrieved using the parameter-surface estimation method.
