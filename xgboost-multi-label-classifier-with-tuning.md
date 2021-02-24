---
description: '#xgboost #parameters #classification #multilabel #f1 #confusionmatrix'
---

# Xgboost multi label classifier with Tuning

Train a multi class classifier with Xgboost tuning the parameters.

This [link](https://xgboost.readthedocs.io/en/latest/parameter.html) contains the complete reference to xgboost parameters and metrics.

```python
import xgboost as xgb
import numpy as np
import pandas as pd
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix
from sklearn.metrics import f1_score


def xgboost_params_tuning(gbmtrain, tune_it, num_classes):
    max_depth = [6, 7, 9, 10, 11, 12]
    min_child_weight = [1, 2, 3, 5, 6, 7, 8]
    subsample = [.6, .7, .8, 1]
    colsample = [.7, .8, .9, 1]
    eta = [.3, .2, .1, .05, .01, .005]
    alpha = [0, 2, 4]
    lambdas = [0, 1, 4, 10]
    results = []

    for i in range(tune_it):
        print(i)
        np.random.seed(i)
        md = max_depth[np.random.randint(len(max_depth) - 1)]
        mcw = min_child_weight[np.random.randint(len(min_child_weight) - 1)]
        ss = subsample[np.random.randint(len(subsample) - 1)]
        cs = colsample[np.random.randint(len(colsample) - 1)]
        et = eta[np.random.randint(len(eta) - 1)]
        al = alpha[np.random.randint(len(alpha) - 1)]
        la = lambdas[np.random.randint(len(lambdas) - 1)]

        params = {'objective': 'multi:softmax', 'nthread': 2, 'seed': 2017, 'verbose_eval': False, 'silent': True,
                  'max_depth': md, 'min_child_weight': mcw, 'subsample': ss, 'colsample': cs, 'eta': et, 'alpha': al,
                  'lambda': la, 'num_class': num_classes}
        cv_results = xgb.cv(
            params,
            gbmtrain,
            num_boost_round=200,
            verbose_eval=False,
            seed=2017,
            nfold=5,
            metrics='merror',
            early_stopping_rounds=10
        )
        print(cv_results)
        results.append([md, mcw, ss, cs, et, al, la] + [cv_results['test-merror-mean'].idxmin() + 1] + list(
            cv_results.loc[cv_results['test-merror-mean'].idxmin()].values))
        print([md, mcw, ss, cs, et, al, la])
        print(cv_results['test-merror-mean'].min())

    columns_df = ['max_depth', 'min_child_weight', 'subsample', 'colsample', 'eta', 'alpha', 'lambda',
                  'iteration', 'test-merror-mean', 'test-merror-std', 'train-merror-mean', 'train-merror-std']
    tuning_results = pd.DataFrame(results, columns=columns_df)
    return tuning_results


def main():
    num_classes = 10
    x, y = make_classification(n_samples=1000, n_features=10, n_redundant=0, n_informative=5,
                               random_state=1, n_clusters_per_class=2, n_classes=num_classes)

    x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=.3, random_state=42)

    gbm_train = xgb.DMatrix(x_train, label=y_train)
    gbm_test = xgb.DMatrix(x_test)

    tuning_results = xgboost_params_tuning(gbm_train, 30, num_classes)
    best_tuning = tuning_results.loc[tuning_results['test-merror-mean'].idxmin()]
    xgbparams = {'objective': 'multi:softmax', 'nthread': 6, 'seed': 2017, 'verbose_eval': 0, 'eval_metric': 'merror',
                 'silent': True, 'max_depth': int(best_tuning['max_depth']),
                 'min_child_weight': int(best_tuning['min_child_weight']), 'subsample': best_tuning['subsample'],
                 'colsample': best_tuning['colsample'], 'eta': best_tuning['eta'], 'alpha': int(best_tuning['alpha']),
                 'lambda': int(best_tuning['lambda']), 'num_class': num_classes}
    num_boost_round = int(best_tuning['iteration'])

    xgbmodel = xgb.train(xgbparams, gbm_train, num_boost_round=num_boost_round)

    var_imp = pd.DataFrame([x for x in xgbmodel.get_score(importance_type='gain').items()],
                           columns=['var', 'gain']).sort_values(by='gain', ascending=False)
    print('Var imp:', var_imp)

    y_predicted = xgbmodel.predict(gbm_test)

    cm = confusion_matrix(y_test, y_predicted)
    f1 = f1_score(y_test, y_predicted, average='weighted')

    print('XGBoost cm:', cm)
    print('XGBoost f1-score:', f1)


if __name__ == '__main__':
    main()

```

Created by: Cosimo Fiorini

