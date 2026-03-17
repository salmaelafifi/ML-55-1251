# Clustering Project — Overall Analysis

---

## Final Comparisons

### Multi-Blob Dataset

The Multi-Blob dataset was generated with 6 known clusters, making it the only dataset 
in this project where the ground truth number of clusters is certain. KMeans with K=6 
achieved the highest silhouette score of 0.4860 among all methods that correctly found 
6 clusters, confirming that KMeans is the most appropriate method for this dataset given 
its spherical, well-separated blob structure. Hierarchical clustering with euclidean+ward 
linkage at K=6 came close with 0.4687, showing that hierarchical methods are competitive 
when the correct cluster count is enforced. However the best raw hierarchical result 
collapsed to 2 clusters with a score of 0.4724, showing that without enforcing K=6 the 
algorithm merges smaller blobs together. DBSCAN performed the weakest on this dataset, 
even its best raw result of 0.2928 corresponds to only 2 clusters, and when forced to 
find 6 clusters the score drops to 0.1420. This is expected: DBSCAN is not well-suited 
for blob data where clusters have varying densities and sizes, as it cannot enforce a 
fixed number of clusters and merges or fragments blobs depending on the eps value. 
Overall KMeans is the clear winner on this dataset.

| Method                              | Silhouette | Clusters |
|-------------------------------------|:----------:|:--------:|
| K-Means (K=6)                       | 0.4860     | 6        |
| Hierarchical (euclidean+ward, K=6)  | 0.4687     | 6        |
| Hierarchical (best raw)             | 0.4724     | 2        |
| DBSCAN (6 clusters)                 | 0.1420     | 6        |
| DBSCAN (best raw)                   | 0.2928     | 2        |

---

### Iris Dataset

The Iris results reveal a consistent and important pattern across all three methods: 
every technique, on both scaled and unscaled data, naturally finds 2 clusters rather 
than the known 3. This is not a failure of implementation but a direct consequence of 
applying unsupervised methods to a dataset where two of the three classes: versicolor 
and virginica, overlap significantly in feature space and cannot be separated by 
geometric or density structure alone. The best overall silhouette scores are 0.6867 for 
unscaled hierarchical and DBSCAN, and 0.6810 for unscaled KMeans, all corresponding to 
2-cluster solutions that cleanly separate setosa from the merged versicolor-virginica 
group. After scaling, all three methods converge to the same silhouette score of 0.5818 
with 2 clusters, showing that scaling produces consistent results across methods but 
does not resolve the overlap problem. When the methods are specifically evaluated at 3 
clusters on scaled data, which is the known ground truth, KMeans achieves the best score 
of 0.4599, Hierarchical comes close at 0.4496, and DBSCAN performs very poorly at 
0.0094, confirming that DBSCAN fundamentally cannot recover the 3-class structure on 
this dataset due to the density overlap between the two similar classes. The 3-cluster 
silhouette scores being lower than the 2-cluster scores is expected and does not indicate 
poor performance, it reflects the genuine ambiguity in the data that only supervised 
methods with access to labels can fully resolve.

| Method                                  | Silhouette | Clusters |
|-----------------------------------------|:----------:|:--------:|
| K-Means (unscaled)                      | 0.6810     | 2        |
| K-Means (scaled)                        | 0.5818     | 2        |
| Hierarchical (unscaled, best overall)   | 0.6867     | 2        |
| Hierarchical (scaled, best overall)     | 0.5818     | 2        |
| DBSCAN (unscaled, best overall)         | 0.6867     | 2        |
| DBSCAN (scaled, best overall)           | 0.5818     | 2        |
| K-Means (scaled, K=3)                   | 0.4599     | 3        |
| Hierarchical (scaled, 3 clusters)       | 0.4496     | 3        |
| DBSCAN (scaled, 3 clusters)             | 0.0094     | 3        |

---

### Customer Dataset

The customer dataset results highlight both the impact of scaling and the importance of 
interpreting silhouette scores alongside cluster counts. On unscaled data, Hierarchical 
clustering achieves the highest score of 0.7293 with 2 clusters, closely followed by 
DBSCAN at 0.7181 also with 2 clusters, and KMeans at 0.5834 also with 2 clusters. 
However, these high unscaled scores are driven primarily by the Income feature dominating 
the distance calculation, meaning the algorithm is effectively splitting customers into 
high-income and low-income groups while largely ignoring the other 6 features. This 
produces a clean but shallow segmentation. After scaling with MinMaxScaler, all features 
contribute equally and the results change significantly. KMeans finds 6 clusters with a 
silhouette of 0.4373, and Hierarchical finds 5 clusters with 0.3993, lower scores but 
far more granular and actionable customer segments. The DBSCAN scaled best overall score 
of 0.6184 with 53 clusters is misleading and should be disregarded, it represents 
micro-clustering at a very low eps value rather than meaningful customer segmentation. 
Overall, the scaled KMeans and Hierarchical results are the most useful for customer 
segmentation as they produce multiple interpretable segments that reflect the combined 
profile of all customer features rather than being dominated by a single high-magnitude 
feature.

| Method                                  | Silhouette | Clusters |
|-----------------------------------------|:----------:|:--------:|
| K-Means (unscaled)                      | 0.5834     | 2        |
| K-Means (scaled)                        | 0.4373     | 6        |
| Hierarchical (unscaled, best overall)   | 0.7293     | 2        |
| Hierarchical (scaled, best overall)     | 0.3993     | 5        |
| DBSCAN (unscaled, best overall)         | 0.7181     | 2        |
| DBSCAN (scaled, best overall)           | 0.6184     | 53       |

---

## Questions

---

### Compare between KMeans, Hierarchical and DBSCAN

KMeans is the simplest and most scalable of the three methods. It requires the number 
of clusters K to be specified in advance, works best with spherical and compact clusters, 
and is sensitive to centroid initialisation, which is addressed by using k-means++ 
initialisation. It does not handle noise or outliers and assumes clusters of roughly 
equal size. On all three datasets KMeans produced stable and consistent results, making 
it the most reliable baseline method. Hierarchical clustering does not require K to be 
specified upfront, the number of clusters emerges from a distance threshold applied to 
the dendrogram. It is more flexible in terms of cluster shapes and provides a visual 
hierarchy of cluster merges through the dendrogram, making it easier to interpret the 
natural grouping structure of the data. However it is computationally expensive and does 
not scale well to very large datasets. On the Iris and Customer datasets, Hierarchical 
clustering achieved the highest raw silhouette scores overall, but these corresponded to 
2-cluster solutions driven by dominant features rather than meaningful multi-class 
separation. DBSCAN requires no cluster count specification and is the only method among 
the three that can identify and isolate noise points, labelling them as -1 rather than 
forcing them into a cluster. It handles arbitrary cluster shapes well and is particularly 
powerful when clusters have clear density separation. However it is highly sensitive to 
the eps and min_samples parameters, requires careful manual calibration to the feature 
scale of each dataset, and struggles when clusters have varying densities or overlapping 
boundaries, both of which were observed in the Iris and Customer datasets. Across all 
three datasets, KMeans was the most consistent performer, Hierarchical produced the 
highest raw scores but often at the cost of over-merging, and DBSCAN was the most 
sensitive to preprocessing and parameter choices.

---

### How did you tune your clustering hyperparameters?

For KMeans, the Elbow method was used to plot distortion against K to identify where 
the rate of improvement slows down, and the silhouette score was plotted against K to 
identify the value where cluster separation is maximised. These two plots together guided 
the selection of the best K for each dataset. The k-means++ initialisation was used 
throughout to ensure stable and reproducible centroid placement, avoiding the poor random 
starts that can occur with standard random initialisation. For Hierarchical clustering, 
multiple combinations of affinity: euclidean, manhattan and cosine, and linkage: ward, 
complete and average, were systematically tested. The dendrogram was used to visually 
identify a natural distance threshold by looking for the largest vertical gap between 
merges, and the silhouette score was computed for each combination to objectively select 
the best configuration. Ward linkage with euclidean affinity consistently performed best 
as it minimises within-cluster variance at each merge step. For DBSCAN, a grid search 
was performed over eps and min_samples ranges that were carefully calibrated to the 
feature scale of each dataset, a standard range of 0.1–3.0 was used for scaled data, 
while much larger ranges were required for unscaled data due to the raw feature 
magnitudes. Three diagnostic plots were produced for each dataset and scaling condition: 
silhouette score vs EPS, silhouette score vs min_samples, and number of clusters vs EPS. 
These three plots together were essential for identifying parameter combinations that 
balance a high silhouette score with a meaningful and interpretable number of clusters, 
since silhouette score alone is insufficient when DBSCAN over-segments the data into 
micro-clusters at very low EPS values.

---

### What is the effect of different distance functions on the calculated clusters?

Euclidean distance measures the straight-line distance between points in feature space 
and works best when clusters are compact and spherical and when all features are on 
comparable scales. It was the most effective metric across all three datasets and all 
clustering methods, particularly after scaling. Manhattan distance sums the absolute 
differences across features rather than squaring them, making it slightly more robust to 
outliers since large individual feature differences are not amplified. On the unscaled 
Iris dendrograms, both Euclidean and Manhattan produced nearly identical 2-cluster 
structures, confirming that the dominant factor at that stage was the unscaled feature 
range rather than the metric choice. After scaling, Euclidean consistently outperformed 
Manhattan in terms of silhouette score and cluster interpretability, as the scaled 
features are well-distributed and the squared differences in Euclidean distance better 
capture the geometric separation between clusters. Cosine distance measures the angle 
between feature vectors rather than their magnitude, making it more appropriate for 
high-dimensional sparse data such as text. On the Iris dataset it produced weaker 
clustering results because the data is inherently spatial, the magnitude of sepal and 
petal measurements carries meaningful biological information that cosine distance 
discards entirely by normalising all vectors to unit length. Overall, the choice of 
distance metric had a measurable but secondary effect on clustering outcomes compared 
to the effect of scaling, scaling consistently produced a larger improvement than 
switching between Euclidean and Manhattan, and cosine distance was consistently the 
weakest performer on all spatial datasets in this project.

---

### How did you evaluate the performance of different clustering techniques?

The primary evaluation metric used throughout the project was the silhouette score, 
which measures how similar each point is to its own cluster compared to the nearest 
neighbouring cluster. It ranges from -1 to 1, where values closer to 1 indicate 
well-separated and compact clusters, values near 0 indicate overlapping clusters, and 
negative values indicate that points may have been assigned to the wrong cluster. For 
each method and dataset, the best silhouette score was stored and compiled into a final 
comparison table covering all techniques on both scaled and unscaled data, allowing a 
direct side-by-side comparison. Visual inspection of cluster plots was used alongside 
the silhouette score as a necessary cross-check, since a high silhouette score alone can 
be misleading, this was particularly evident for DBSCAN on the Customer dataset where 
micro-clustering at low EPS produced a score of 0.6184 with 53 meaningless clusters, 
and for all methods on the Iris and Customer unscaled data where 2-cluster solutions 
scored highly but represented shallow, feature-dominated segmentation rather than genuine 
structure. For the Iris dataset specifically, the cluster assignments were also compared 
against the known true labels after clustering to assess how well the unsupervised 
methods recovered the natural 3-class structure, providing an additional layer of 
evaluation not possible for the Customer dataset where no ground truth labels exist. The 
combination of silhouette score, cluster count, visual inspection and where available 
label comparison provided a robust and multi-dimensional evaluation framework throughout 
the project.

---

### What is the effect of scaling on the performance of clustering techniques?

Scaling had a significant and consistent impact across all three clustering methods and 
all datasets in this project. Without scaling, features with larger absolute ranges 
dominate the distance calculation entirely — on the Customer dataset, Income ranging 
from 35,832 to 309,364 overwhelms all other features including Age, Education, 
Occupation and Settlement Size, causing all three methods to effectively cluster on 
Income alone and produce trivial 2-cluster high-income versus low-income splits with 
artificially high silhouette scores. For KMeans, scaling produced more balanced and 
granular clusters, the Customer dataset went from 2 clusters unscaled to 6 clusters 
scaled, reflecting a richer and more actionable segmentation of the customer base across 
all features. For Hierarchical clustering, scaling changed the dendrogram structure 
meaningfully, on the Iris dataset, the scaled Euclidean dendrogram was the only 
configuration that revealed a hint of 3-cluster structure, while all unscaled 
configurations collapsed to 2 clusters. For DBSCAN, scaling was the most critical factor 
of all: on the Customer dataset, the standard eps range of 0.1–3.0 returned absolutely 
no valid clusters on unscaled data because the raw inter-point distances are orders of 
magnitude larger, requiring a completely different eps range of 100–10,000 for the 
unscaled version. After MinMaxScaler, a standard eps range of 0.1–1.0 became immediately 
applicable and produced interpretable results. The overall conclusion is that scaling is 
not an optional preprocessing step for distance-based clustering, it is a fundamental 
requirement when features operate on different scales, and its absence leads to biased, 
shallow and potentially misleading clustering outcomes regardless of which algorithm is 
used.