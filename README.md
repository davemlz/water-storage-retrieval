# Water Storage Retrieval

Semi-supervised water storage retrieval using Sentinel-2 and *in situ* bathymetric data.

The pipeline is divided into three phases:

## Phase I: Automatic Water Classification

Water is automatically classificated in an object-based approach using Simple Non-Iterative Clustering (SNIC) and k-means.

First, water is automatically classified in `/phase-I/01-automatic-water-classification.ipynb`. The Sentinel-2 image is preprocessed (median filtering is applied) and segmented using SNIC algorithm in Google Earth Engine. As inputs for the segmentation, the 10 m bands + the NDWI were used. Later, the segmented image is clustered using k-means. The cluster with the higher NDWI centroid is selected as the water cluster. The remaining clusters are labeled as non-water clusters.

Multiple hyperparameters were tested: Seed spacing and grid type for the SNIC algorithm, and the number of clusters and training size for the k-means algorithm. The results for each combination of hyperparameters were compared against actual water masks and statistical analysis were performef in `/phase-I/02-statistical-analysis.Rmd`.

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

## Phase II: Satellite Derieved Bathymetry

Water depth is estimated through 3 different regression models: Linear Regression, Random Forest and Gradient Boosting.

## Phase III: Water Storage Retrieval

Total water storage volume is retrieved using the parameter-surface estimation method.
