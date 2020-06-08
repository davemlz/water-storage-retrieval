# Water Storage Retrieval

Semi-supervised water storage retrieval using Sentinel-2 and *in situ* bathymetric data.

The pipeline is divided into three phases:

## Phase I: Automatic Water Classification

Water is automatically classificated in an object-based approach using Simple Non-Iterative Clustering (SNIC) and k-means.

First, water is automatically classified in `/phase-I/01-automatic-water-classification.ipynb`. The Sentinel-2 image is preprocessed (median filtering is applied) and segmented using SNIC algorithm in Google Earth Engine. As inputs for the segmentation, the 10 m bands + the NDWI were used. Later, the segmented image is clustered using k-means. The cluster with the higher NDWI centroid is selected as the water cluster. The remaining clusters are labeled as non-water clusters.

Multiple hyperparameters were tested: Seed spacing and grid type for the SNIC algorithm, and the number of clusters and training size for the k-means algorithm. The results for each combination of hyperparameters were compared against actual water masks and statistical analysis were performed in `/phase-I/02-statistical-analysis.Rmd`.

For an automatic water mask, use the `automaticWaterMask` function in `/phase-I/01-automatic-water-classification.ipynb`.

```python
automaticWaterMask(image, # Preprocessed Sentinel-2 image
                   ROI, # Region of Interest
                   index, # Index to use (NDWI or GNDVI)
                   seedSpacing, # Seed spacing for SNIC in pixels. 10 pixels is recommended
                   gridType, # Grid type for SNIC ("square" or "hex"). "square" is recommended.
                   scale, # Spatial resolution to work with. 10 for Sentinel-2.
                   k, # Number of clusters for k-means. 4 is recommended.
                   pTrain) # Training size as a fraction of the number of superpixels. 2 is recommended.
```

## Phase II: Satellite Derived Bathymetry

Water depth is estimated through 3 different regression models: Linear Regression, Random Forest and Gradient Boosting.

First, reservoirs shorelines are retrieved and added to the original bathymetric data in `/phase-II/01-shoreline-to-bathymetry.ipynb`. The gathered data is interpolated using IDW in `/phase-II/02-IDW-interpolation.Rmd`. The input features for the regression models are calculated in `/phase-II/03-input-features.ipynb`. Regression models were fitted an tested in `/phase-II/04-bathymetry-estimation.ipynb`. Predictions were also made in this notebook. Statistical analysis of cross-validation results from the previous step were performed in `/phase-II/05-statistical-analysis.Rmd`.

### Input Features

The input features for the regression models were the 10 m bands + Lyzenga Transforms + Ratio Transforms + Cumulative Cost Map.

Lyzenga Transform can be computed as the log transform of an specific band: `log(x - xs)`, where `x` is the reflectance of the band x and `xs` is the average reflectance of deep waters in band x. For lakes and reservoirs, `xs` is set to zero.

Ratio Transform is calculated as the ratio of two log transformed bands: `log(n*x)/log(n*y)`, where `x` is the reflectance of the band x, `y` is the reflectance of the band y and `n` is an user defined parameter. Here, `n` was set to 1.

The most important input feature for bathymetry estimation is the cumulative cost. This feature represents the euclidean distance from each pixel inside a water mask to the closest vertex in the shoreline of the water mask. Cumulative cost can be computed using the `depthCumulativeCost` function in `/phase-II/03-input-features.ipynb`.

```python
depthCumulativeCost(waterMask, # Water mask (water mask derived from automaticWaterMask can be used here).
                    ROI, # Region of interest.
                    scale, # Spatial resolution to work with. 10 for Sentinel-2.
                    maxDistance) # Max distance to look for a closest point. 1000 is recommended.
```

### Ensembles

Ensembles methods performed better than Linear Regression in bathymetry estimation. Random Forest and Gradient Boosting can be used. Best hyperparameters are showed below.

1. Random Forest

```python
from sklearn.ensemble import RandomForestRegressor
RandomForestRegressor(n_estimators = 20,
                      max_features = "sqrt")
```
2. Gradient Boosting

```python
from sklearn.ensemble import GradientBoostingRegressor
GradientBoostingRegressor(n_estimators = 500,
                          learning_rate = 0.1,
                          max_depth = 20,
                          max_features = "sqrt",
                          validation_fraction = 0.05, # Can be changed depending on your training size.
                          n_iter_no_change = 10,
                          tol = 0.01)
```

## Phase III: Water Storage Retrieval

Total water storage volume is retrieved using the parameter-surface estimation method in `/phase-III/01-water-storage-retrieval.ipynb`. Bathymetry predictions of the three regression models were tested. Ensembles methods showed a better performance for water storage retrieval (RMSE < 4 hm^3).
