# Unsupervised Nutritional Profiling of USDA Foods Using K-Means Clustering
**Evaluating Nutrient-Based Food Classification Independent of Commercial Marketing Labels**

---
## Executive Summary
Analyzed 7,058 USDA food items across 7 nutrient vectors using Principal Component Analysis (PCA) and K-Means clustering ($K=4$, Silhouette Score = 0.4158). The model successfully grouped foods by true biological nutrient density, placing items like plant-based sausage and dried walrus meat in the same high-protein cluster, proving that unsupervised machine learning can objectively classify foods independent of commercial marketing labels.

---
## Abstract
This project applies unsupervised machine learning to 7,058 USDA food items to identify nutrient-based groupings independent of commercial labeling. By analyzing foods based solely on quantitative nutrient composition, this study explores whether computational methods can reveal patterns that are not immediately apparent from traditional food categories or commercial marketing descriptions. Using standardized nutrient features, Principal Component Analysis (PCA), and K-Means clustering, the analysis identifies patterns in how foods organize according to their observed nutrient profiles.

**This document is an executive summary.** For the full statistical methodology (including the complete hyperparameter search, multiple independent cluster-validity metrics, correlation analysis, and an explicit discussion of where the model's fit is only moderate rather than clean), see [TECHNICAL_REPORT.md](TECHNICAL_REPORT.md). For a plain-language version with no statistics background required, see [NONTECHNICAL_EXPLAINER.md](NONTECHNICAL_EXPLAINER.md). To try the model yourself, open the [live Food Cluster Explorer](https://lucaritivoi.github.io/usda-nutritional-phenotypes-kmeans/food_cluster_explorer.html): search any of the 7,058 foods, or enter custom nutrient values to see which cluster they'd fall into, live.

---

## 1. Project Overview & Objective
Nutritional marketing often emphasizes specific claims (e.g., "low fat" or "diet") that may not fully represent a product's overall nutritional profile. The objective of this study is to evaluate whether an unsupervised machine learning model can objectively categorize food products based strictly on their quantitative nutrient composition, independent of commercial marketing labels.

---

## 2. Repository & Project Structure
```text
.
├── USDA_Clustering_Analysis.ipynb   # Main Jupyter Notebook with data pipeline & visualizations
├── USDA.csv                         # Source dataset (7,058 USDA food items, 16 nutrient fields)
├── cluster_visualizations.png       # Generated PCA scatter plot and centroid heatmap
├── README.md                        # Executive summary (this file)
├── TECHNICAL_REPORT.md              # Full technical report: methodology, statistics, limitations
├── NONTECHNICAL_EXPLAINER.md        # Plain-language version, no statistics background required
├── food_cluster_explorer.html       # Interactive tool: search foods or classify a custom one, live
└── requirements.txt                 # Python environment dependencies
```

---

## 3. Dataset & Methodology
This study analyzed **7,058 food items** from the USDA National Nutrient Database across seven core nutrient features (per 100g serving):
* **Calories (kcal)**
* **Protein (g)**
* **Total Fat (g)**
* **Saturated Fat (g)**
* **Carbohydrates (g)**
* **Sugar (g)**
* **Sodium (mg)**

### Machine Learning Pipeline
* **Data Normalization:** Features were standardized using a Z-score `StandardScaler` to prevent high-magnitude features (e.g., sodium in mg) from disproportionately influencing Euclidean distance calculations.
* **Dimensionality Reduction:** Principal Component Analysis (PCA) was applied to project the 7-dimensional feature space onto two principal axes (PC1 and PC2), retaining **62.35%** of overall feature variance (PC1: 35.50%, PC2: 26.85%).
* **Clustering & Hyperparameter Optimization:** K-Means clustering was executed for K values from 1 through 10 (`n_init=10` for deterministic convergence across scikit-learn versions). Hyperparameter selection (K = 4) was guided by Within-Cluster Sum of Squares (WCSS) inflection and validated via Silhouette Analysis, yielding an overall **Silhouette Score of 0.4158** (indicating moderate cluster separation).

---

## 4. Results & Visualizations

![Cluster Visualizations](cluster_visualizations.png?v=2)

### Cluster Profile Summary (Mean Values per 100g)

Cluster IDs are not meaningful on their own (K-Means assigns them arbitrarily during fitting), so clusters are relabeled 0→3 in order of increasing mean Calories immediately after fitting. This keeps the numbering stable and interpretable across reruns, rather than depending on initialization order or library version.

| Cluster | Profile Description | Key Characteristics | Representative Examples | Size (n, %) |
| :--- | :--- | :--- | :--- | :--- |
| **Cluster 0** | **Low Energy Density** | Low caloric and macronutrient concentration, high water content | Raw vegetables, leafy greens, simple broth bases | 2,647 (37.5%) |
| **Cluster 1** | **High Protein** | High protein content with variable fat profiles | Poultry, seafood, lean meats, plant protein isolates | 2,660 (37.7%) |
| **Cluster 2** | **High Carbohydrate / Sodium Profile** | Elevated sugars, starches, and sodium levels | Processed grains, baked goods, snacks, confectioneries | 1,523 (21.6%) |
| **Cluster 3** | **Lipid-Dense** | Concentrated fats and high energy density | Plant oils, animal fats, shortenings, nut butters | 228 (3.2%) |

### Key Observations:
* **Convergence Across Biological Origins:** High-protein items clustered together regardless of source origin. For instance, dried walrus meat and plant-based sausage alternatives mapped to the same high-protein cluster based strictly on nutrient composition.
* **Nutritional Similarity Among Differently Marketed Products:** Some reduced-fat and diet-labeled products clustered alongside traditional high-carbohydrate processed foods, suggesting that single marketing attributes may not fully represent overall nutrient profiles.

---

## 5. Reproducibility & Code Execution
The dataset (`USDA.csv`) is included directly in this repository, so the analysis can be reproduced end-to-end with no external downloads (Python 3.9+ recommended):

1. Clone this repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
   ```
2. Install required dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Launch Jupyter Notebook and execute `USDA_Clustering_Analysis.ipynb` from top to bottom.

Results are fully deterministic (`random_state=42`, `n_init=10`) and should reproduce the exact Silhouette Score and variance-explained figures reported above.

---

## 6. Limitations & Future Work

### Limitations
* **Geometric Assumptions:** K-Means assumes isotropic (spherical) cluster geometry, which may not capture complex non-linear boundaries in nutritional data.
* **Feature Scope:** The model focused primarily on macronutrients and selected nutritional variables. Important factors such as dietary fiber, essential vitamins, and minerals were not included and may influence broader nutritional classification.

### Future Directions
* Application of density-based models (e.g., DBSCAN) or hierarchical clustering to detect non-spherical sub-clusters.
* Expansion of feature vectors to include dietary fiber and key micronutrient profiles.

---

## 7. Conclusion
These results demonstrate that unsupervised clustering can identify meaningful groupings of foods based on nutrient composition. This approach provides a transparent, data-driven framework for comparing nutrient-based classifications with commercial marketing categories.

---

## 8. References
* **Dataset used in this analysis:** USDA National Nutrient Database (7,058 items, 16 fields), as distributed via MIT 15.071 *The Analytics Edge* (Spring 2017) course materials, MIT OpenCourseWare. https://www.ocw.mit.edu/courses/15-071-the-analytics-edge-spring-2017/resources/usda/
* **U.S. Department of Agriculture, Agricultural Research Service.** FoodData Central, 2019. https://fdc.nal.usda.gov/
* **Pedregosa et al.** *Scikit-learn: Machine Learning in Python*. Journal of Machine Learning Research (JMLR), 12, pp. 2825-2830, 2011.
* **Jolliffe, I. T.** *Principal Component Analysis*. Springer Series in Statistics, Springer-Verlag, 2002.
* **MacQueen, J.** *Some methods for classification and analysis of multivariate observations.* Proceedings of the 5th Berkeley Symposium on Mathematical Statistics and Probability, 1967.
