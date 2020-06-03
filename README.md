# Water Storage Retrieval

Reservoirs water storage retrieval using Sentinel-2.

The pipeline is divided into three phases:

## Phase I: Automatic Water Classification

Water is automatically classificated in an object-based approach. Simple Non-Iterative Clustering and k-means are applied here.

## Phase II: Satellite Derieved Bathymetry

Water depth is estimated through 3 different regression models: Linear Regression, Random Forest and Gradient Boosting.

## Phase III: Water Storage Retrieval

Total water storage volume is retrieved using the parameter-surface estimation method.
