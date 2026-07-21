# Unsupervised Nutritional Profiling of USDA Foods Using K-Means Clustering
**Evaluating Objective Macro and Micronutrient Geometry to Identify Commercial Labeling Discrepancies**

---

## 1. Project Overview & Objective
Nutritional marketing often emphasizes specific claims (e.g., "low fat" or "diet") that may obscure a product's overall metabolic profile. The objective of this study is to evaluate whether an unsupervised machine learning model can objectively categorize food products based strictly on their chemical composition, independent of commercial marketing labels.

---

## 2. Dataset & Methodology
This study analyzed **7,058 food items** from the USDA National Nutrient Database across seven core quantitative nutritional features (per 100g serving):
* **Calories (kcal)**
* **Protein (g)**
* **Total Fat (g)**
* **Saturated Fat (g)**
* **Carbohydrates (g)**
* **Sugar (g)**
* **Sodium (mg)**

### Machine Learning Pipeline
* **Data Normalization:** Features were standardized using a $Z$-score `StandardScaler` to prevent high-magnitude features (e.g., sodium in mg) from disproportionately influencing Euclidean distance calculations.
* **Dimensionality Reduction:** Principal Component Analysis (PCA) was applied to project the 7-dimensional feature space onto two principal axes ($PC1$ and $PC2$), retaining **59.12%** of overall feature variance.
* **Clustering & Hyperparameter Optimization:** $K$-Means clustering was executed for $K \in [1, 10]$. Hyperparameter selection ($K = 4$) was guided by Within-Cluster Sum of Squares (WCSS) inflection and validated via Silhouette Analysis, yielding an overall **Silhouette Score of 0.4014**.

---

## 3. Results & Visualizations

![Cluster Visualizations](cluster_visualizations.png)

The algorithm grouped the dataset into four distinct nutritional phenotypes based on feature centroids:

1. **Cluster 0 (Low-Density / Baseline):** Minimal macro-nutrient density; primarily raw vegetables and hydration-dense items.
2. **Cluster 1 (High Carbohydrate & High Sodium):** Heavily processed grains, snack foods, and refined sweets.
3. **Cluster 2 (Lipid-Dense):** Concentrated fat sources, including plant oils, animal fats, and shortenings.
4. **Cluster 3 (High Protein):** Dense protein sources, including meats, poultry, seafood, legumes, and protein isolates.

### Key Observations:
* **Convergence Across Biological Origins:** High-protein items clustered together regardless of source origin. For instance, dried walrus meat and plant-based sausage alternatives mapped to the same high-protein cluster based strictly on macronutrient density.
* **Marketing Overlap in Processed Foods:** "Diet" or "reduced-fat" packaged items (e.g., reduced-fat biscuits and snack novelties) clustered directly within the High-Carbohydrate/High-Sodium profile alongside traditional confectioneries, indicating that fat reduction often coincides with elevated carbohydrate and sodium levels.

---

## 4. Limitations & Future Work

### Limitations
* **Geometric Assumptions:** $K$-Means assumes isotropic (spherical) cluster geometry, which may not capture complex non-linear boundaries in nutritional data.
* **Feature Scope:** The model excluded micronutrients (vitamins, minerals) and dietary fiber, which significantly impact glycemic response and overall nutritional quality.

### Future Directions
* Application of density-based models (e.g., DBSCAN) or hierarchical clustering to detect non-spherical sub-clusters.
* Expansion of feature vectors to include dietary fiber and key micronutrients.

---

## 5. Conclusion
These results demonstrate that unsupervised clustering can effectively group foods by objective chemical composition, providing a transparent, data-driven framework for evaluating food profiles alongside commercial claims.
