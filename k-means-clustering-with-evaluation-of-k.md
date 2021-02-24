---
description: '#kmeans #unsupervised #clustering'
---

# K-Means clustering with evaluation of k

Evaluation of multiple k choices for a k-means clustering algorithm. The choice of the correct numbert of cluster can be guided by the two metrics defined.

The [Calinski Harabaz score](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.calinski_harabasz_score.html) is defined as ratio between the within-cluster dispersion and the between-cluster dispersion and must be maximized.

The Total Within SS becomes zeros at the limit of k=\#observation. The choice should be the last k that gives a significant decay of the number. AIC and BIC consider also the number k letting the curve grow up when the number of cluster grows. The choice can be then the k that correspond to the minimum of the curve.



![First analysis to choose  k - In this specific case the number choosed for k should be 7](.gitbook/assets/image%20%281%29.png)

```python
import numpy as np
import pandas as pd
from sklearn.datasets import make_blobs
from sklearn import metrics
from sklearn import cluster
import matplotlib.pyplot as plt


def diagnostics_from_clusteringmodel(model, len_db):
    total_within_sumofsquares = model.inertia_
    number_of_clusters = len(model.cluster_centers_)
    number_of_dimensions = len(model.cluster_centers_[0])
    number_of_rows = len_db

    aic = total_within_sumofsquares + 2 * number_of_dimensions * number_of_clusters
    bic = total_within_sumofsquares + np.log(number_of_rows) * number_of_dimensions * number_of_clusters

    return {'Clusters': number_of_clusters,
            'Total Within SS': total_within_sumofsquares,
            'AIC': aic,
            'BIC': bic}


def main():
    n_samples = 1500
    random_state = 170
    x, y = make_blobs(n_samples=n_samples, random_state=random_state, centers=7)
    max_n_clusters = 50
    kmeans = [cluster.KMeans(n_clusters=i, n_jobs=-1).fit(x) for i in range(2, max_n_clusters)]

    diagnostics = pd.DataFrame([diagnostics_from_clusteringmodel(model, len(x)) for model in kmeans])
    diagnostics.set_index('Clusters', inplace=True)
    plt.figure()
    diagnostics.plot(kind='line')
    plt.savefig('C:/WORK/diag_p.png')

    ch_score = [metrics.calinski_harabasz_score(x, model.labels_) for model in kmeans]
    plt.figure()
    plt.plot(range(2, max_n_clusters), ch_score)
    plt.savefig('C:/WORK/ch_eval.png')


if __name__ == '__main__':
    main()

```

Created by: Cosimo Fiorini

