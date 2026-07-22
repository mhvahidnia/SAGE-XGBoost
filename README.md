# SAGE-XGBoost

### Spatially Augmented Graph-Embedded XGBoost (NOTEBOOK: SAGE-XGBoost.ipynb)

This repository contains the official implementation of **SAGE-XGBoost**, a machine-learning framework designed to improve susceptibility mapping when only limited labeled samples are available. The framework combines:

* Controlled data augmentation
* K-nearest-neighbor (KNN) graph feature extraction
* PCA-based graph embedding
* Spatial coordinate encoding
* XGBoost classifier

The implementation is provided as a Google Colab notebook.

---

# Required Input Data

The model requires the following datasets.

## 1. Environmental raster layers

Prepare all conditioning factors as GeoTIFF files having

* identical coordinate reference system (CRS)
* identical spatial resolution
* identical spatial extent

Example:

```python
tif_paths = {
    "Elevation": ".../elevation.tif",
    "Slope": ".../slope.tif",
    "Aspect": ".../aspect.tif",
    ...
}
```

Only this dictionary needs to be modified for a new study area.

---

## 2. Hazard inventory

Prepare the hazard inventory as a point shapefile.

Example:

```python
fire_gdf = gpd.read_file(
    ".../inventory.shp"
)
```

The shapefile must contain

* Point geometry
* One attribute containing susceptibility class labels

Example

| Geometry | Intensity |
| -------- | --------- |
| Point    | 1         |
| Point    | 2         |
| Point    | 3         |

The attribute name should be updated here:

```python
fire_gdf["Intensity"]
```

---

## 3. Number of susceptibility classes

Current implementation uses 3 hazard classes from the intensity field and 1 non-hazard class based on the random selection

```python
num_class = 4
```

If another number of classes is used, update

* inventory labels
* XGBoost parameter

```python
num_class = ...
```

---

# Parameters That Can Be Modified

## Data augmentation

```python
USE_AUGMENTATION = True

AUG_RATIO = 0.90

NOISE_LEVEL = 0.04
```

| Parameter        | Description                                       |
| ---------------- | ------------------------------------------------- |
| USE_AUGMENTATION | Enable/disable augmentation                       |
| AUG_RATIO        | Percentage of additional training samples         |
| NOISE_LEVEL      | Noise standard deviation (fraction of feature SD) |

---

## KNN Graph

```python
K = 15
```

Number of neighboring samples used for graph construction.

Typical values

```
5–20
```

---

## PCA Embedding

```python
EMB_DIM = 3
```

Number of graph embedding dimensions.

---

## Train-Test Split

```python
test_size = 0.20
```

Default split:

* Training = 70%
* Testing = 30%

---

## XGBoost Hyperparameters

Search ranges are defined here:

```python
xgb_params = {

    "n_estimators":[100,200,300,400],

    "max_depth":[4,6],

    "learning_rate":[0.05,0.1],

    "subsample":[0.8,1.0],

    "colsample_bytree":[0.8]
}
```

These ranges can be freely adjusted.

---

# Workflow

The implementation follows the following pipeline.

```
Raster Layers
      │
Inventory
      │
      ▼
Rasterization
      │
Sample Extraction
      │
Train/Test Split
      │
Data Augmentation (Training Only)
      │
KNN Graph Construction
      │
Graph Feature Extraction
      │
PCA Embedding
      │
Coordinate Normalization
      │
Feature Fusion
      │
GridSearchCV
      │
Final XGBoost Model
      │
Prediction
      │
Evaluation
```

---

# Outputs

The notebook reports

* Best hyperparameters
* Training time
* Peak memory usage
* Confusion matrix
* Classification report
* Accuracy
* Weighted F1-score
* ROC-AUC
* PCA explained variance
* PCA loading matrix

---

# Notes

* Data augmentation is applied **only to the training set** to prevent information leakage.
* PCA is fitted **only on the training graph features**.
* Test graph features are computed using neighbors from the training graph.
* The implementation supports any number of environmental raster layers.

---

# Citation

If you use this implementation in your research, please cite:

#### *Vahidnia, M.H. (2026). SAGE-XGBoost: Spatially Augmented Graph Embeddings–Machine Learning Framework for Natural Hazards Susceptibility Mapping under Data Scarcity. Earth Science Informatics. (Submitted)*

---

# License

MIT License
