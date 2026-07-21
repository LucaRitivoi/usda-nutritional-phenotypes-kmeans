# Unsupervised Machine Learning Classification of Nutritional Phenotypes
**Evaluating Macronutrient Geometry and Marketing Bias via K-Means Clustering on USDA Food Profiles**

---

## Executive Summary
Commercial food labeling frequently obscures objective nutritional composition and metabolic impact through consumer-facing marketing strategies. To isolate genuine nutritional profiles, a multi-dimensional feature space was constructed from $N = 7,058$ entries in the USDA National Nutrient Database, tracking six core parameters: Calories, Total Fat, Saturated Fat, Carbohydrates, Sugars, and Sodium. 

Optimization via the Elbow Method established $K=4$ as the foundational macro-phenotypic structure for a K-Means clustering algorithm utilizing Euclidean distance metrics. Unsupervised classification successfully isolated four distinct food profiles: a low-calorie baseline (predominantly raw vegetables), a lipid-dense group (oils and pure fats), a high-protein cohort, and a high-carbohydrate/high-sodium category representing ultra-processed foods. 

Crucially, the model bypassed labeling nomenclature to classify items purely by chemical geometry, grouping dense protein sources indiscriminately of origin while dynamically segregating low-protein meat variants based on sub-surface macronutrient deficiency. This study demonstrates that unsupervised machine learning can effectively strip away marketing bias to expose the mathematical health geometry of commercial food supplies.

---

## 1. Introduction & Research Question
Modern nutritional messaging relies heavily on consumer-facing descriptors such as *"low-fat," "diet,"* or *"protein-enriched."* However, these qualitative labels often fail to capture the quantitative realities of a product's metabolic impact. Because FDA labeling guidelines allow room for strategic branding, products with vastly different biological profiles can share similar health claims, while products with identical chemical structures are marketed under completely divergent categories.

### Key Hypotheses:
1. **Unsupervised Classification:** An unsupervised K-Means clustering algorithm, operating strictly on standardized quantitative macronutrient features, will ignore commercial naming conventions and group foods based entirely on underlying biochemical geometry.
2. **Detection of Marketing Anomalies:** The algorithm will expose structural overlaps between commercially marketed *"diet/health"* foods and traditional ultra-processed, high-sugar/high-sodium items.

---

## 2. Data Preprocessing & Mathematical Methodology

### Dataset Architecture
The primary dataset was extracted from the **USDA National Nutrient Database** ($N = 7,058$), targeting six high-impact macronutrient and micronutrient parameters per 100g serving:
* **Protein (g)**
* **Total Fat (g)**
* **Saturated Fat (g)**
* **Carbohydrates (g)**
* **Sugar (g)**
* **Sodium (mg)**

### Feature Engineering & Data Normalization
To prevent features with larger raw magnitudes (such as Sodium in milligrams) from artificially dominating Euclidean distance calculations over lower-magnitude features (such as Protein in grams), feature variance was standardized using **Z-score normalization** ($StandardScaler$):

$$z = \frac{x - \mu}{\sigma}$$

*Where $x$ is the raw feature value, $\mu$ is the feature mean, and $\sigma$ is the standard deviation.* Missing values across nutrient features were statically imputed using column-mean substitution to maintain sample size integrity without shifting global distributions.

### Optimization via the Elbow Method
To determine the optimal number of clusters ($K$), the **Within-Cluster Sum of Squares (WCSS)** was evaluated across $K \in [1, 10]$:

$$WCSS = \sum_{k=1}^{K} \sum_{x_i \in C_k} ||x_i - \mu_k||^2$$

Evaluating the inflection point (or "elbow") on the WCSS curve revealed structural bends at $K=4$ and $K=6$. $K=4$ was chosen as the foundational macro-phenotypic structure to maximize inter-cluster variance while avoiding over-segmentation.
