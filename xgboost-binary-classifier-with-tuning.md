---
description: '#xgboost #parameters #binaryclassification #auc'
---

# Xgboost binary classifier with Tuning

Train a binary classifier with Xgboost tuning the parameters. The code is specific for an unbalanced case: the unbalance factor deal with the problem. Furthermore, tha auc is choosen as metric because of the unbalancing. Change it in case of balanced classes. 

This [link](https://xgboost.readthedocs.io/en/latest/parameter.html) contains the complete reference to xgboost parameters and metrics.

```python
import xgboost as xgb
import numpy as np
import pandas as pd
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_curve, auc


def xgboost_params_tuning(gbmtrain, tune_it, unbalance):
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

        params = {'objective': 'binary:logistic', 'nthread': 2, 'seed': 2017, 'verbose_eval': False, 'silent': True,
                  'max_depth': md, 'min_child_weight': mcw, 'subsample': ss, 'colsample': cs, 'eta': et, 'alpha': al,
                  'lambda': la, 'scale_pos_weight': unbalance}
        cv_results = xgb.cv(
            params,
            gbmtrain,
            num_boost_round=200,
            verbose_eval=False,
            seed=2017,
            nfold=5,
            metrics='auc',
            early_stopping_rounds=10
        )
        print(cv_results)
        results.append([md, mcw, ss, cs, et, al, la] + [cv_results['test-auc-mean'].idxmax() + 1] + list(
            cv_results.loc[cv_results['test-auc-mean'].idxmax()].values))
        print([md, mcw, ss, cs, et, al, la])
        print(cv_results['test-auc-mean'].max())

    columns_df = ['max_depth', 'min_child_weight', 'subsample', 'colsample', 'eta', 'alpha', 'lambda',
                  'iteration', 'test-auc-mean', 'test-auc-std', 'train-auc-mean', 'train-auc-std']
    tuning_results = pd.DataFrame(results, columns=columns_df)
    return tuning_results


def main():
    x, y = make_classification(n_samples=1000, n_features=10, n_redundant=0, n_informative=5,
                               random_state=1, n_clusters_per_class=4)

    x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=.3, random_state=42)

    # in case of unbalanced classes, unbalance value can be used by the model
    unbalance = (len(y_train) - y_train.sum()) / y_train.sum()

    gbm_train = xgb.DMatrix(x_train, label=y_train)
    gbm_test = xgb.DMatrix(x_test)

    tuning_results = xgboost_params_tuning(gbm_train, 30, unbalance)
    best_tuning = tuning_results.loc[tuning_results['test-auc-mean'].idxmax()]
    xgbparams = {'objective': 'binary:logistic', 'nthread': 6, 'seed': 2017, 'verbose_eval': 0, 'eval_metric': 'auc',
                 'silent': True, 'max_depth': int(best_tuning['max_depth']),
                 'min_child_weight': int(best_tuning['min_child_weight']), 'subsample': best_tuning['subsample'],
                 'colsample': best_tuning['colsample'], 'eta': best_tuning['eta'], 'alpha': int(best_tuning['alpha']),
                 'lambda': int(best_tuning['lambda']), 'scale_pos_weight': unbalance}
    num_boost_round = int(best_tuning['iteration'])

    xgbmodel = xgb.train(xgbparams, gbm_train, num_boost_round=num_boost_round)

    var_imp = pd.DataFrame([x for x in xgbmodel.get_score(importance_type='gain').items()],
                           columns=['var', 'gain']).sort_values(by='gain', ascending=False)
    print('Var imp:', var_imp)

    y_predicted = xgbmodel.predict(gbm_test)

    fpr, tpr, trhs = roc_curve(list(y_test), list(y_predicted))
    roc_auc = auc(fpr, tpr)
    print('XGBoost auc:', roc_auc)


if __name__ == '__main__':
    main()

```

## Details

Let's have a look to the code.

### Main function

First part create a sample dataset. You should substitute it with your data.

```python
x, y = make_classification(n_samples=1000, n_features=10, n_redundant=0, n_informative=5,
                               random_state=1, n_clusters_per_class=4)

x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=.3, random_state=42)

```

The matrices are transformed in DMatrix to be used by xgboost. If you have the names of the predictors you could add the list as _feature\_names_

```python
gbm_train = xgb.DMatrix(x_train, label=y_train)
gbm_test = xgb.DMatrix(x_test)
```

The tuning function is called and the best params are selected. The number 30 in line 1 is the number of random searches we want to be performed in the grid of the parameters.

```python
tuning_results = xgboost_params_tuning(gbm_train, 30, unbalance)
best_tuning = tuning_results.loc[tuning_results['test-auc-mean'].idxmax()]
xgbparams = {'objective': 'binary:logistic', 'nthread': 6, 'seed': 2017, 'verbose_eval': 0, 'eval_metric': 'auc',
                 'silent': True, 'max_depth': int(best_tuning['max_depth']),
                 'min_child_weight': int(best_tuning['min_child_weight']), 'subsample': best_tuning['subsample'],
                 'colsample': best_tuning['colsample'], 'eta': best_tuning['eta'], 'alpha': int(best_tuning['alpha']),
                 'lambda': int(best_tuning['lambda']), 'scale_pos_weight': unbalance}
num_boost_round = int(best_tuning['iteration'])
```

Once the model is trained we can identify the most important features.

```python
var_imp = pd.DataFrame([x for x in xgbmodel.get_score(importance_type='gain').items()],
                           columns=['var', 'gain']).sort_values(by='gain', ascending=False)
print('Var imp:', var_imp)

```

And finally evaluate the auc metric for the test set.

```python
y_predicted = xgbmodel.predict(gbm_test)

fpr, tpr, trhs = roc_curve(list(y_test), list(y_predicted))
roc_auc = auc(fpr, tpr)
print('XGBoost auc:', roc_auc)
```

### Tuning function

Possible grid of the parameters.

```python
max_depth = [6, 7, 9, 10, 11, 12]
min_child_weight = [1, 2, 3, 5, 6, 7, 8]
subsample = [.6, .7, .8, 1]
colsample = [.7, .8, .9, 1]
eta = [.3, .2, .1, .05, .01, .005]
alpha = [0, 2, 4]
lambdas = [0, 1, 4, 10]
```

Random selection for each iteration of a params list

```python
np.random.seed(i)
md = max_depth[np.random.randint(len(max_depth) - 1)]
mcw = min_child_weight[np.random.randint(len(min_child_weight) - 1)]
ss = subsample[np.random.randint(len(subsample) - 1)]
cs = colsample[np.random.randint(len(colsample) - 1)]
et = eta[np.random.randint(len(eta) - 1)]
al = alpha[np.random.randint(len(alpha) - 1)]
la = lambdas[np.random.randint(len(lambdas) - 1)]

params = {'objective': 'binary:logistic', 'nthread': 2, 'seed': 2017, 'verbose_eval': False, 'silent': True,
          'max_depth': md, 'min_child_weight': mcw, 'subsample': ss, 'colsample': cs, 'eta': et, 'alpha': al,
          'lambda': la, 'scale_pos_weight': unbalance}
```

Cross validation with early stopping setted to 10.

```python
cv_results = xgb.cv(
            params,
            gbmtrain,
            num_boost_round=200,
            verbose_eval=False,
            seed=2017,
            nfold=5,
            metrics='auc',
            early_stopping_rounds=10
        )
```

Created by: Cosimo Fiorini

