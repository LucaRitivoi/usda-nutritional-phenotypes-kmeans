# Technical Report: Unsupervised Nutritional Profiling of USDA Foods Using K-Means Clustering

**Author:** Milu
**Companion document:** [README.md](README.md) (executive summary) · [USDA_Clustering_Analysis.ipynb](USDA_Clustering_Analysis.ipynb) (full code)

---

## Abstract

This report presents an analysis of an unsupervised machine learning model applied to 7,058 food items from the USDA National Nutrient Database, built around the idea that food should be classifiable by its actual nutrient composition rather than by marketing claims or popular health narratives. Seven nutrient features per 100g serving (Calories, Protein, Total Fat, Saturated Fat, Carbohydrate, Sugar, and Sodium) were standardized and clustered using K-Means (K = 4, Silhouette Score = 0.4158, Davies-Bouldin Index = 0.9052, Calinski-Harabasz Index = 2800.23). Where the companion README summarizes the project for a general audience, this report documents the full methodology, the reasoning behind each modeling decision, and, equally important, the points where the model's fit is only moderate rather than clean, so the analysis's actual strength and actual limits are both stated plainly rather than implied.

---

## 1. Introduction

This project comes out of something that's bothered me for a while: how little most people actually understand about the nuances of health and nutrition, and how much marketing and conflicting information get in the way of forming an accurate picture. My first instinct was that the fix was for people to simply know more, to get better at understanding nutrition science themselves. But the more I thought about it, the more that felt backwards: needing to become an expert simply to defend yourself against misleading marketing is a symptom of the problem, not a solution to it. What's actually missing is an objective way to look at food that doesn't depend on marketing claims, personal opinion, or whatever health trend happens to be popular. Something that simply describes what a food objectively is, nutritionally, without anyone's spin attached.

This project asks a narrower, answerable version of that question: if you strip away every label and marketing claim and look only at a food's quantitative nutrient composition, what groupings does it fall into on its own? K-Means clustering is a natural tool for this because it partitions data purely by geometric proximity in feature space, with no access to category labels, brand information, or human-assigned food groups.

The purpose of this report is to document that pipeline with the level of statistical detail a technical reader would expect: the exact preprocessing decisions and their justification, the full hyperparameter search rather than simply the chosen value, multiple independent cluster-validity metrics rather than one, and an honest accounting of where the model's assumptions are strained by the data.

---

## 2. Data

### 2.1 Source

The dataset used is the USDA National Nutrient Database as distributed via the MIT 15.071 *The Analytics Edge* course materials (MIT OpenCourseWare, Spring 2017): 7,058 food items across 16 fields, of which 7 nutrient fields (plus `Description` as an identifier) are used in this analysis. Full citation in [Section 9](#9-references).

### 2.2 Feature Selection

Seven macronutrient-level features were selected: **Calories, Protein, Total Fat, Saturated Fat, Carbohydrate, Sugar, Sodium** (all per 100g). These were chosen because they are populated across nearly the entire dataset, are measured on a consistent, comparable basis (per 100g), and correspond to the attributes most commonly cited in commercial nutrition marketing, which is directly relevant to the project's motivating question. Micronutrients (Calcium, Iron, Potassium, Vitamin C, Vitamin E, Vitamin D) are present in the source data but excluded here; see [Section 7](#7-limitations) for the rationale and the cost of that exclusion.

### 2.3 Missing Data

Missingness is uneven across features and is not trivial for all of them:

| Feature | Missing (n) | Missing (%) |
| :--- | ---: | ---: |
| Calories | 1 | 0.01% |
| Protein | 1 | 0.01% |
| Total Fat | 1 | 0.01% |
| Saturated Fat | 301 | 4.26% |
| Carbohydrate | 1 | 0.01% |
| **Sugar** | **1,910** | **27.06%** |
| Sodium | 84 | 1.19% |

Sugar was missing for 27% of the 7,058 foods, far more than any other feature, and all of those missing values were filled in with the dataset's average sugar content. This is a real limitation, not simply a technical footnote: when that many points get set to the exact same value, the true spread of sugar content across those foods disappears. K-means groups foods by how different they are from each other, so flattening a quarter of the data toward one number on this dimension can blur the boundaries of whichever cluster depends most on sugar, in this case, the high-carbohydrate group. This is treated in more depth in [Section 7](#7-limitations).

### 2.4 Descriptive Statistics (post-imputation, raw units, per 100g)

| Feature | Mean | Std Dev | Min | 25th pct | Median | 75th pct | Max |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Calories | 219.70 | 172.19 | 0.0 | 85.00 | 181.00 | 331.00 | 902.00 |
| Protein (g) | 11.71 | 10.92 | 0.0 | 2.29 | 8.20 | 20.43 | 88.32 |
| Total Fat (g) | 10.32 | 16.81 | 0.0 | 0.72 | 4.38 | 12.70 | 100.00 |
| Saturated Fat (g) | 3.45 | 6.77 | 0.0 | 0.20 | 1.41 | 3.81 | 95.60 |
| Carbohydrate (g) | 20.70 | 27.63 | 0.0 | 0.00 | 7.13 | 28.17 | 100.00 |
| Sugar (g) | 8.26 | 13.12 | 0.0 | 0.20 | 4.82 | 8.26 | 99.80 |
| Sodium (mg) | 322.06 | 1039.18 | 0.0 | 38.00 | 80.00 | 383.00 | 38,758.00 |

Two things stand out immediately. First, every feature has a standard deviation on the same order as (or larger than) its mean. These are right-skewed distributions, not roughly-normal ones, which is expected for nutrient data (most foods are unremarkable on any given axis; a few are extreme). Second, Sodium's standard deviation (1,039.18) is more than three times its mean (322.06), driven by a max of 38,758 mg/100g (a small number of extremely sodium-dense items like bouillon or salt substitutes). This matters directly for the modeling choices in [Section 3](#3-methodology).

### 2.5 Feature Correlation

| | Calories | Protein | Total Fat | Sat. Fat | Carb | Sugar | Sodium |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **Calories** | 1.00 | 0.14 | 0.81 | 0.60 | 0.43 | 0.27 | 0.03 |
| **Protein** | 0.14 | 1.00 | 0.06 | 0.04 | -0.28 | -0.25 | -0.00 |
| **Total Fat** | 0.81 | 0.06 | 1.00 | 0.75 | -0.11 | -0.05 | 0.00 |
| **Sat. Fat** | 0.60 | 0.04 | 0.75 | 1.00 | -0.11 | 0.01 | -0.01 |
| **Carb** | 0.43 | -0.28 | -0.11 | -0.11 | 1.00 | 0.61 | 0.05 |
| **Sugar** | 0.27 | -0.25 | -0.05 | 0.01 | 0.61 | 1.00 | -0.01 |
| **Sodium** | 0.03 | -0.00 | 0.00 | -0.01 | 0.05 | -0.01 | 1.00 |

Calories correlates strongly with Total Fat (r = 0.81) and Saturated Fat (r = 0.60), which makes sense energetically: fat carries roughly twice the calories per gram of protein or carbohydrate. Carbohydrate and Sugar also correlate moderately (r = 0.61), since sugar is a subset of total carbohydrate. This matters because K-Means operates on Euclidean distance in the full 7-dimensional standardized space: correlated features do not each contribute independent information, so the "energy-related" cluster of features (Calories, Total Fat, Saturated Fat) has more combined influence on distance than its 3-of-7 share would suggest. Sodium, by contrast, is essentially uncorrelated with everything else (|r| ≤ 0.05 across the board). It behaves as an independent axis of variation, which is consistent with it loading almost entirely onto neither principal component (see [Section 3.6](#36-dimensionality-reduction-pca)).

---

## 3. Methodology

### 3.1 Standardization

Features were standardized to zero mean and unit variance using `sklearn.preprocessing.StandardScaler` (Z-score standardization):

$$z = \frac{x - \mu}{\sigma}$$

This step was necessary because the seven features weren't on comparable scales: Sodium, for example, is recorded in milligrams and ranges into the thousands (0–38,758), while Protein is recorded in grams and tops out under 100 (0–88.32). Since K-means clusters points based on distance, a feature with a much larger numeric range will dominate that distance calculation even if it isn't actually more nutritionally significant. The algorithm has no way to know that Sodium's bigger numbers are simply a byproduct of the unit it's measured in. Standardizing every feature to the same scale (mean 0, standard deviation 1) means each nutrient contributes to clustering based on how unusual it is relative to other foods, not on which unit it happened to be measured in.

### 3.2 K-Means Formulation

K-Means partitions *n* observations into *K* clusters by minimizing the within-cluster sum of squared distances to each cluster's centroid:

$$\underset{S}{\arg\min} \sum_{i=1}^{K} \sum_{x \in S_i} \lVert x - \mu_i \rVert^2$$

where $S_i$ is the set of points assigned to cluster *i* and $\mu_i$ is that cluster's centroid. K-means requires you to specify K in advance: it has no built-in way of deciding how many groups is "right"; that's what the elbow method ([Section 3.3](#33-hyperparameter-selection-the-elbow-method)) is for. Once K is fixed, the algorithm drops K starting points into the data (centroids) and repeats a two-step process: every food gets assigned to whichever centroid it's currently closest to, then each centroid moves to the average position of all the foods just assigned to it. This repeats (reassign, then re-average) until foods stop switching groups and the centroids settle (formally, this is Lloyd's algorithm). Because the objective is non-convex, where those starting centroids land matters; this analysis uses `k-means++` initialization (which spreads initial centroids apart probabilistically rather than placing them uniformly at random) with `n_init=10`, meaning the algorithm is run 10 times from different starting points and the best result (lowest WCSS) is kept. `random_state=42` fixes the randomness so results are exactly reproducible.

One consequence worth stating explicitly: **the integer cluster labels K-Means outputs (0, 1, 2, 3, ...) carry no inherent meaning or ordering**: they are assigned arbitrarily based on where the starting centroids happened to land, and are not guaranteed to be stable across runs, library versions, or even `n_init` values. This matters for reproducibility specifically: if the cluster numbers could shift between runs, someone rerunning this notebook to verify the analysis would see different foods under the same cluster number than what this report describes, making it look like the analysis doesn't hold up, when really the underlying groups are identical and only an arbitrary label changed. Relabeling clusters by ascending mean Calories right after fitting fixes the numbering to something stable and meaningful, so "Cluster 0" refers to the same real group every time, for anyone who reruns the code (see [Section 4.1](#41-cluster-profiles)).

### 3.3 Hyperparameter Selection: The Elbow Method

K-Means requires K to be chosen in advance. WCSS (inertia) was computed for K = 1 through 10:

| K | WCSS |
| ---: | ---: |
| 1 | 49,406.0 |
| 2 | 37,286.8 |
| 3 | 28,946.2 |
| **4** | **22,550.4** |
| 5 | 17,710.8 |
| 6 | 14,591.9 |
| 7 | 12,343.4 |
| 8 | 10,690.6 |
| 9 | 9,839.8 |
| 10 | 9,196.2 |

WCSS always goes down as K increases. Pushed to the extreme, K equal to the number of foods (7,058) would give a WCSS of exactly 0, since every "cluster" would simply be one food by itself. That's technically the lowest possible WCSS, but it's meaningless: nothing has actually been grouped. So choosing K isn't about finding the lowest WCSS; it's about finding the point where adding another cluster stops capturing a real, substantial split in the data and starts simply carving up groups that were already reasonably tight. K=1 through K=4 each produce a large drop in the table above (49,406 → 37,287 → 28,946 → 22,550). Each new cluster is finding a real distinction. After K=4, the drops get smaller relative to the added complexity of one more group to interpret. That slowdown is the "elbow," and it is the basis for selecting K=4 instead of a higher value that would technically have lower WCSS but would simply be fragmenting the data without adding real meaning.

### 3.4 Cluster Validation: Silhouette Analysis

The Silhouette Score measures, for each point, how much closer it is to its own cluster than to the nearest neighboring cluster (ranging from -1 to 1; higher is better-separated). Silhouette was computed for every K from 2 to 10, not only K = 4:

| K | Silhouette Score |
| ---: | ---: |
| 2 | 0.4346 |
| 3 | 0.4472 |
| **4** | **0.4158** |
| 5 | 0.4234 |
| 6 | 0.4434 |
| 7 | 0.4560 |
| 8 | 0.4558 |
| 9 | 0.4622 |
| 10 | 0.4408 |

This table shows a real tradeoff, and it's worth reading carefully rather than skipping to the conclusion: silhouette score is actually higher at K=3 (0.4472) and highest in the K=7–9 range (up to 0.4622 at K=9). If silhouette score were the only criterion, K=9 would be the "better" choice. K=4 was chosen instead because that gap is small, and the cost of chasing it is real: going from 4 clusters to 9 means splitting the data into far more specific categories, which stops being useful past a certain point. Four clusters map cleanly onto nameable, understandable nutritional profiles (see [Section 4.1](#41-cluster-profiles)); nine clusters would fragment those same categories into finer subdivisions that are harder to characterize and communicate, for a marginal statistical gain. This is a deliberate tradeoff between statistical tightness and interpretability, not an oversight; it is stated here explicitly rather than presenting K=4 as though it were the unambiguous optimum on every metric.

### 3.5 Additional Validation Metrics (K = 4)

Two further metrics, independent of silhouette, were computed to cross-check the K = 4 solution: the same logic as running multiple trials in an experiment: no single measurement is fully trustworthy on its own, but if three independently-computed metrics all point the same direction, that agreement is more convincing than any one of them alone.

- **Davies-Bouldin Index: 0.9052** (lower is better; measures average similarity between each cluster and its most-similar other cluster). A value under 1 indicates the clusters are, on average, more distinct from their nearest neighbor than they are internally spread out.
- **Calinski-Harabasz Index: 2,800.23** (higher is better; ratio of between-cluster to within-cluster dispersion). There is no fixed universal threshold for this metric, so it is most useful for comparing candidate values of K against each other rather than as a standalone number, included here for completeness and to support future comparison if K is revisited.

### 3.6 Dimensionality Reduction (PCA)

The clustering itself uses all 7 standardized features, but a scatter plot only has 2 axes. Simply picking 2 of the 7 raw features (say, Calories and Protein) to plot would throw away the other 5 entirely and risk missing the direction along which clusters actually separate. Principal Component Analysis avoids that by looking at all 7 features at once and mathematically constructing 2 new axes, each a blend of all 7 original features, chosen specifically to capture as much of the total spread in the data as possible:

| Component | Variance Explained | Cumulative |
| :--- | ---: | ---: |
| PC1 | 35.50% | 35.50% |
| PC2 | 26.85% | 62.35% |
| PC3 | 14.34% | 76.69% |
| PC4 | 12.78% | 89.47% |
| PC5 | 6.65% | 96.12% |
| PC6 | 3.80% | 99.92% |
| PC7 | 0.08% | 100.00% |

PC1 and PC2 together retain 62.35% of total variance, enough to make the 2D scatter plot in [Figure 1](cluster_visualizations.png) a reasonably faithful (though not complete) representation of cluster separation in the full 7-dimensional space. The near-zero variance on PC7 (0.08%) indicates the 7 original features are only about 6-dimensionally independent, consistent with the correlation structure noted in [Section 2.5](#25-feature-correlation).

The component loadings clarify what PC1 and PC2 physically represent:

| Feature | PC1 loading | PC2 loading |
| :--- | ---: | ---: |
| Calories | 0.595 | 0.107 |
| Protein | 0.042 | -0.382 |
| Total Fat | 0.574 | -0.213 |
| Saturated Fat | 0.521 | -0.202 |
| Carbohydrate | 0.146 | 0.638 |
| Sugar | 0.145 | 0.591 |
| Sodium | 0.017 | 0.037 |

**PC1** loads heavily and positively on Calories, Total Fat, and Saturated Fat, the three features that make a food calorie-dense, so it reads as an *energy-density axis*. **PC2** shows a composition split: high in Carbohydrate and Sugar sits on one end, high in Protein sits on the opposite end, the two pulling in opposite directions along the same axis: a *carbohydrate-vs-protein composition axis*, not a health judgment; a sugary snack and a fiber-rich starch can both sit at the same end of this axis despite being very different foods, because the axis measures composition, not health. Sodium loads weakly on both components (0.017, 0.037), consistent with its near-zero correlation with every other feature ([Section 2.5](#25-feature-correlation)). It is a largely independent axis of variation that this particular 2D projection doesn't capture well, even though it is a full, standardized input to the clustering itself.

---

## 4. Results

### 4.1 Cluster Profiles

Clusters are numbered 0→3 in order of increasing mean Calories (see [Section 3.2](#32-k-means-formulation)):

| Cluster | Name | n | % of total | Calories | Protein (g) | Total Fat (g) | Sat. Fat (g) | Carb (g) | Sugar (g) | Sodium (mg) | Mean Silhouette |
| :--- | :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | Low Energy Density | 2,647 | 37.50% | 76.33 ± 55.21 | 3.05 ± 3.19 | 1.63 ± 2.84 | 0.77 ± 1.34 | 12.53 ± 9.85 | 5.80 ± 4.93 | 228.61 ± 494.18 | 0.6116 |
| 1 | High Protein | 2,660 | 37.69% | 224.52 ± 102.10 | 23.24 ± 8.08 | 13.22 ± 10.90 | 4.42 ± 3.80 | 2.71 ± 7.38 | 2.94 ± 3.85 | 291.65 ± 464.84 | 0.3447 |
| 2 | High Carbohydrate / Sodium | 1,523 | 21.58% | 380.05 ± 77.86 | 7.92 ± 5.32 | 9.53 ± 10.10 | 3.07 ± 4.50 | 68.78 ± 14.98 | 22.60 ± 21.33 | 560.40 ± 2028.30 | 0.2086 |
| 3 | Lipid-Dense | 228 | 3.23% | 756.65 ± 146.27 | 3.11 ± 5.39 | 82.59 ± 19.71 | 25.85 ± 22.35 | 4.16 ± 10.00 | 2.97 ± 5.76 | 169.76 ± 305.42 | 0.3564 |

(Values are mean ± standard deviation per 100g, computed within each cluster.)

The per-cluster mean silhouette column is worth reading alongside the profile means: Cluster 0 (Low Energy Density) is the best-separated cluster by a wide margin (0.6116), while Cluster 2 (High Carbohydrate/Sodium) is the weakest (0.2086), meaning a meaningful fraction of Cluster 2's members sit close to the boundary with a neighboring cluster rather than deep in its interior. This is discussed further in [Section 6](#6-discussion).

### 4.2 Qualitative Validation

Beyond the summary statistics, individual item inspection supports the same groupings the model found numerically. Cluster 1 (High Protein) groups items as biologically and commercially distinct as dried walrus meat and plant-based sausage alternatives together, purely on nutrient composition, despite one being a traditional animal product and the other explicitly marketed as an alternative to one. Sampling 10 random items from Cluster 1 (High Protein) surfaces almost exclusively meats and a plant-protein outlier (peanut butter, itself protein-dense at ~25g/100g); sampling Cluster 2 (High Carbohydrate/Sodium) surfaces candies, cereals, and toaster pastries, items that share little in common by traditional food category but converge on the same nutrient profile. Full sample lists are in the notebook's Section 7 output.

### 4.3 Visualization

![Cluster Visualizations](cluster_visualizations.png)

**Figure 1** (left) shows the 2D PCA projection, colored by final cluster assignment. This view captures only 62.35% of total variance, so apparent closeness between clusters here does not necessarily reflect their actual separation in the full 7-dimensional standardized space where the silhouette scores are computed. Cluster 0 (Low Energy Density, red) sits visually close to Cluster 1 (High Protein, blue) near the origin, yet Cluster 0 has the highest per-cluster silhouette of all four groups (0.6116, [Section 4.1](#41-cluster-profiles)), separation that this 2D projection alone does not show. Cluster 3 (Lipid-Dense, purple) extends furthest along PC1, consistent with its extreme mean Calories and Total Fat. **Figure 2** (right) is the centroid heatmap underlying the table in [Section 4.1](#41-cluster-profiles).

---

## 5. Reproducibility

This analysis is fully deterministic: `random_state=42` fixes K-Means initialization, `n_init=10` is set explicitly (scikit-learn's default `n_init` behavior has changed across versions, leaving it implicit is a real, if subtle, reproducibility risk), and package versions are pinned in `requirements.txt`. Running `USDA_Clustering_Analysis.ipynb` top-to-bottom against the included `USDA.csv` should reproduce every number in this report exactly.

---

## 6. Discussion

The headline number, Silhouette Score 0.4158, sits in the "reasonable, moderate structure" range (roughly 0.3–0.5 is typical for real-world nutrient/behavioral data; scores above 0.7 are rare outside synthetic or highly separable datasets). That moderate score is not a flaw to explain away; it is an accurate reflection of the underlying data. Food nutrient composition is continuous, not naturally clustered into crisp, well-separated categories: a slice of pizza sits genuinely between "high carbohydrate" and "high fat," and the model correctly reflects that ambiguity as boundary-region uncertainty rather than forcing a false sharp distinction.

The per-cluster silhouette breakdown in [Section 4.1](#41-cluster-profiles) makes this concrete: Cluster 2 (High Carbohydrate/Sodium) has the weakest internal cohesion (0.2086), and it is also the cluster with the largest within-cluster Sodium standard deviation (2,028.30, larger than its own mean of 560.40). That combination is not a coincidence. StandardScaler changes the *scale* of a feature (putting everything in comparable units), but it does not change the *shape* of its distribution. Sodium was heavily right-skewed in the raw data ([Section 2.4](#24-descriptive-statistics-post-imputation-raw-units-per-100g)), and that skew persists after scaling; it is simply measured in standard deviations instead of milligrams. So when a handful of extremely salty foods (likely cured, brined, or heavily processed items) land in Cluster 2 alongside more typically-salted foods (both grouped there because they are similarly high in carbohydrate and sugar), those outliers still sit far from the rest of their own cluster along the Sodium dimension, even after standardization. That is precisely what a low silhouette score captures: foods that do not sit cleanly inside their assigned group. The cluster with the largest internal Sodium spread being the same cluster with the weakest silhouette score is not a coincidence: one is a direct, traceable cause of the other.

The K-vs-silhouette table in [Section 3.4](#34-cluster-validation-silhouette-analysis) is included specifically so a reader does not have to take K = 4 on faith: the data show a genuine trade-off between statistical tightness (favoring K = 9) and interpretability (favoring a smaller K), and this report names that trade-off explicitly rather than picking the number that produces the cleanest-sounding narrative.

---

## 7. Limitations

- **Mean imputation, especially for Sugar.** With 27.06% of Sugar values imputed at the column mean, the true variance in Sugar is understated for over a quarter of the dataset, and any item whose real sugar content was unusually high or low is misrepresented by that imputation for clustering purposes. A more defensive approach (e.g., excluding rows with imputed Sugar from cluster-profile summary statistics, or using a food-category-conditional imputation instead of a single global mean) was not implemented here and is the single highest-value improvement identified for future work.
- **Geometric (isotropic) cluster assumption.** K-Means implicitly assumes clusters are roughly spherical in the standardized feature space. Sodium's extreme right-skew (Section 2.4) is a poor match for that assumption even after Z-score standardization, since standardization corrects scale but not skew.
- **Feature scope.** Only 7 of the dataset's 16 available fields were used. Fiber and micronutrients (Calcium, Iron, Potassium, Vitamins C/E/D) were excluded to keep the analysis focused on the macronutrient attributes most visible in commercial marketing claims, at the cost of a fuller nutritional picture.
- **K selection is a judgment call, not a unique optimum.** As shown in Section 3.4, K = 4 is not the silhouette-maximizing choice; it was selected for elbow-method support plus interpretability. A reader who weights statistical tightness more heavily than interpretability could reasonably argue for a different K, and that disagreement would be legitimate.
- **Dataset coverage and age.** The dataset reflects an older USDA release, and it simply does not include every modern food format. Greek yogurt is a clear example: there is no Greek yogurt anywhere in the 7,058 items, only plain, fruit, and frozen yogurt, so every yogurt entry has protein in the 2.4-5.7g per 100g range, closer to Cluster 0's (Low Energy Density) average protein of 3.05g than to Cluster 1's (High Protein) average of 23.24g. That places all yogurt in Cluster 0, which is the correct output given what the dataset contains, not a model error. Greek yogurt's much higher protein content would very likely place it in Cluster 1 instead, but that is a gap in the data, not a gap in the method.

## 8. Future Directions

- Re-run the pipeline with Sugar-imputed rows excluded from summary statistics (though not necessarily from clustering itself) to quantify how much the imputation affects the reported cluster profiles.
- Apply a log or Box-Cox transform to Sodium (and possibly Calories, Total Fat) before standardizing, to address the skew noted in Section 7, and compare resulting silhouette/Davies-Bouldin scores against the current model.
- Expand the feature set to include Fiber and key micronutrients, and evaluate whether cluster structure changes meaningfully.
- Compare K-Means against a density-based method (DBSCAN) or Gaussian Mixture Models, which do not assume spherical clusters and may fit Sodium's skew better.
- Re-examine K = 7–9 solutions directly (not simply their silhouette scores) to check whether the additional clusters correspond to interpretable sub-categories (e.g., splitting "High Carbohydrate/Sodium" into separate sugar-dominant and sodium-dominant groups) that would justify the added complexity.

## 9. Conclusion

Across four independent validity checks (the elbow method, Silhouette Analysis across the full range of K, the Davies-Bouldin Index, and the Calinski-Harabasz Index), a 4-cluster solution produces a moderately well-separated, interpretable grouping of USDA food items based purely on nutrient composition, with no commercial label or marketing claim involved anywhere in the process. The clusters correspond to recognizable profiles (Low Energy Density, High Protein, High Carbohydrate/Sodium, Lipid-Dense) that group foods across traditional category lines: dried walrus meat and plant-based sausage end up in the same cluster, for example, because they're nutritionally similar, regardless of how differently they're marketed. The model's fit is moderate, not perfect, and this report says exactly where and why it's weakest (Cluster 2's cohesion, driven by Sodium's skew) instead of glossing over it. That's the actual point of this project: an objective, quantitative way to look at food doesn't require anyone, reader or author, to already be a nutrition expert or to trust whatever claim is on the package. It simply requires being honest about what the numbers actually show, including their limits.

---

## 10. References

- **Dataset used in this analysis:** USDA National Nutrient Database (7,058 items, 16 fields), as distributed via MIT 15.071 *The Analytics Edge* (Spring 2017) course materials, MIT OpenCourseWare. https://www.ocw.mit.edu/courses/15-071-the-analytics-edge-spring-2017/resources/usda/
- U.S. Department of Agriculture, Agricultural Research Service. *FoodData Central*, 2019. https://fdc.nal.usda.gov/
- Pedregosa, F., et al. "Scikit-learn: Machine Learning in Python." *Journal of Machine Learning Research*, 12, pp. 2825–2830, 2011.
- Jolliffe, I. T. *Principal Component Analysis*. Springer Series in Statistics, Springer-Verlag, 2002.
- MacQueen, J. "Some methods for classification and analysis of multivariate observations." *Proceedings of the 5th Berkeley Symposium on Mathematical Statistics and Probability*, 1967.
- Rousseeuw, P. J. "Silhouettes: A graphical aid to the interpretation and validation of cluster analysis." *Journal of Computational and Applied Mathematics*, 20, pp. 53–65, 1987.
- Davies, D. L., and Bouldin, D. W. "A Cluster Separation Measure." *IEEE Transactions on Pattern Analysis and Machine Intelligence*, PAMI-1(2), pp. 224–227, 1979.
- Caliński, T., and Harabasz, J. "A dendrite method for cluster analysis." *Communications in Statistics*, 3(1), pp. 1–27, 1974.
