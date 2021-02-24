---
description: '#plotly #confusionmatrix #classificationresults'
---

# Plotly Confusion Matrix

Example of usage of plot\_heatmap function \([https://app.gitbook.com/@energyway/s/models-and-solutions-templates/datavis-miscellanea/plotly\_heatmap\_function](https://app.gitbook.com/@energyway/s/models-and-solutions-templates/datavis-miscellanea/plotly_heatmap_function)\) to show a confusion matrix for a classification model.

## Output and Full Code​

![](../.gitbook/assets/newplot%20%282%29.png)

![](../.gitbook/assets/newplot-1-%20%281%29.png)

```python
import pandas as pd, numpy as np, xgboost as xgb
import plotly.figure_factory as ff

from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix, f1_score


def plot_heatmap(df_heatmap, title='Heatmap', xaxis_title='', yaxis_title='', n_colors=1, figsize=500, filename=''):
    """
    Description

    Parameters
    ----------
    df_heatmap : pandas DataFrame
    title : string, default 'Heatmap'
        Plot title
    xaxis_title : string, default ''
        Name of the x axis
    yaxis_title : string, default ''
        Name of the y axis
    n_colors : {1, 2, 3}, default 1
        Number of colors to use in the Heatmap
    figsize : int, default 500
        Dimension of the square image (figsize, figsize)
    filename : string, default ''
        Name of the filaname. If filename != '' then the image 
        is saved with the filename specified. 
        Supported formats:
        - png (static image)
        - html (interactive image)

    Returns
    -------
    Plotly image
    """
    if n_colors == 1:
        colorscale = [[0, 'white'], [1, 'green']]
        font_colors = ['black', 'black']
    elif n_colors == 2:
        colorscale = [[0, 'lightblue'], [1, 'red']]
        font_colors = ['black', 'black']
    elif n_colors == 3:
        colorscale = [[0, 'lightblue'], [.5, 'white'], [1, 'red']]
        font_colors = ['black', 'black', 'black']
    fig = ff.create_annotated_heatmap(np.round(df_heatmap.values[::-1],2), 
                                      x = list(df_heatmap.columns),
                                      y = list(df_heatmap.index)[::-1],
                                      colorscale=colorscale, font_colors=font_colors)


    fig.update_layout(
        autosize=False,
        width=figsize,
        height=figsize,
        title={'text':title,'x':.5,'y':.99},
        xaxis_title=xaxis_title,
        yaxis_title=yaxis_title,
    )

    if filename.endswith('.html'):
        fig.write_html(filename)
    elif filename.endswith('.png'):
        fig.write_image(filename)

    fig.show()





data = load_wine()
X, Y = data['data'], data['target']
target_names = data['target_names']
X = pd.DataFrame(X,columns=data['feature_names'])
df = X.copy()
df['wine'] = Y

X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.3)



model = xgb.XGBClassifier()
model = model.fit(X_train, Y_train)

P_test = model.predict(X_test)



df_conf = pd.DataFrame(confusion_matrix(Y_test, P_test), columns=target_names, index=target_names)
d = np.concatenate([df_conf.sum(0).values.reshape(1,-1), np.array(df_conf.sum(0).sum()).reshape((1,1))],1)

df_tp = pd.DataFrame([[df_conf.loc[t,t] for t in target_names]], index=['True predict'], columns=target_names)
df_tp['Total'] = df_tp.sum().sum()
df_acc = 100 * df_tp / d
df_acc.rename(index={'True predict':'Accuracy<br>rate %'}, inplace=True)


df_fp = pd.DataFrame([[df_conf.loc[set(target_names)-set([t]),t].sum() for t in target_names]], index=['False predict'], columns=target_names)
df_fp['Total'] = df_fp.sum().sum()

df_fpr = 100 * df_fp / d
df_fpr.rename(index={'False predict':'False predict<br>rate %'}, inplace=True)

df_f1 = 100 * pd.DataFrame(f1_score(Y_test, P_test, average=None).reshape(1,-1), index=['F1 %'], columns=target_names)
df_f1['Total'] = 100 * f1_score(Y_test, P_test, average='weighted')

df_res = pd.concat([df_tp, df_acc, df_fp, df_fpr, df_f1], 0, sort=False)



plot_heatmap(df_conf, title='Confusion matrix', figsize=400, xaxis_title='Predicted class', yaxis_title='Real class')

plot_heatmap(df_res,title='Classification results', xaxis_title='Predicted class', yaxis_title='Prediction metric')
```

## Details

Plot a classification model results in terms of confusion matrix and some classical classification performance metric for each class.

### Step by step procedure

* Step 1. Import libraries

  ```python
  import plotly.figure_factory as ff
  import pandas as pd, numpy as np, xgboost as xgb

  from sklearn.datasets import load_wine
  from sklearn.model_selection import train_test_split

  from sklearn.metrics import confusion_matrix, f1_score
  ```

* Step 2. Define plot\_heatmap funcion, as described in  
  [https://app.gitbook.com/@energyway/s/models-and-solutions-templates/datavis-miscellanea/plotly\_heatmap\_function](https://app.gitbook.com/@energyway/s/models-and-solutions-templates/datavis-miscellanea/plotly_heatmap_function)

  ```python
  def plot_heatmap(df_heatmap, title='Heatmap', xaxis_title='', yaxis_title='', n_colors=1, figsize=500, filename=''):
      """
      Description

      Parameters
      ----------
      df_heatmap : pandas DataFrame
      title : string, default 'Heatmap'
          Plot title
      xaxis_title : string, default ''
          Name of the x axis
      yaxis_title : string, default ''
          Name of the y axis
      n_colors : {1, 2, 3}, default 1
          Number of colors to use in the Heatmap
      figsize : int, default 500
          Dimension of the square image (figsize, figsize)
      filename : string, default ''
          Name of the filaname. If filename != '' then the image 
          is saved with the filename specified. 
          Supported formats:
          - png (static image)
          - html (interactive image)

      Returns
      -------
      Plotly image
      """
      if n_colors == 1:
          colorscale = [[0, 'white'], [1, 'green']]
          font_colors = ['black', 'black']
      elif n_colors == 2:
          colorscale = [[0, 'lightblue'], [1, 'red']]
          font_colors = ['black', 'black']
      elif n_colors == 3:
          colorscale = [[0, 'lightblue'], [.5, 'white'], [1, 'red']]
          font_colors = ['black', 'black', 'black']
      fig = ff.create_annotated_heatmap(np.round(df_heatmap.values[::-1],2), 
                                        x = list(df_heatmap.columns),
                                        y = list(df_heatmap.index)[::-1],
                                        colorscale=colorscale, font_colors=font_colors)


      fig.update_layout(
          autosize=False,
          width=figsize,
          height=figsize,
          title={'text':title,'x':.5,'y':.99},
          xaxis_title=xaxis_title,
          yaxis_title=yaxis_title,
      )

      if filename.endswith('.html'):
          fig.write_html(filename)
      elif filename.endswith('.png'):
          fig.write_image(filename)

      fig.show()
  ```

* Step 3. Load sample data from sklearn \(multiclass classification problem\), create the DataFrame and split data randomly into train \(70%\) and test \(30%\)

  ```python
  data = load_wine()
  X, Y = data['data'], data['target']
  target_names = data['target_names']
  X = pd.DataFrame(X,columns=data['feature_names'])
  Y = pd.Series(Y, name='wine')
  df = pd.concat([X,Y],1)

  X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.3)
  ```

* Step 4. Train a classification model \(here, xgboost default classifier\) and predict test outputs \(P\_test\).

  ```python
  model = xgb.XGBClassifier()
  model = model.fit(X_train, Y_train)

  P_test = model.predict(X_test)
  ```

* Step 5. Compute confusion matrix and put it into a DataFrame: df\_conf. Then compute true predict \(df\_tp\), accuracy \(df\_acc\), false predict \(df\_fp\), false predict rate \(df\_fpr\), F1 score for each predicted class. Note that false predict means wrong prediction for the selected class. All these metrics are collected in df\_res DataFrame.

  ```python
  df_conf = pd.DataFrame(confusion_matrix(Y_test, P_test), columns=target_names, index=target_names)
  d = np.concatenate([df_conf.sum(0).values.reshape(1,-1), np.array(df_conf.sum(0).sum()).reshape((1,1))],1)

  df_tp = pd.DataFrame([[df_conf.loc[t,t] for t in target_names]], index=['True predict'], columns=target_names)
  df_tp['Total'] = df_tp.sum().sum()
  df_acc = 100 * df_tp / d
  df_acc.rename(index={'True predict':'Accuracy<br>rate %'}, inplace=True)


  df_fp = pd.DataFrame([[df_conf.loc[set(target_names)-set([t]),t].sum() for t in target_names]], index=['False predict'], columns=target_names)
  df_fp['Total'] = df_fp.sum().sum()

  df_fpr = 100 * df_fp / d
  df_fpr.rename(index={'False predict':'False predict<br>rate %'}, inplace=True)

  df_f1 = 100 * pd.DataFrame(f1_score(Y_test, P_test, average=None).reshape(1,-1), index=['F1 %'], columns=target_names)
  df_f1['Total'] = 100 * f1_score(Y_test, P_test, average='weighted')

  df_res = pd.concat([df_tp, df_acc, df_fp, df_fpr, df_f1], 0, sort=False)
  ```

* Step 6. Show the confusion matrix using the plot\_heatmap function.

  ```python
  plot_heatmap(df_conf, title='Confusion matrix', figsize=400, 
      xaxis_title='Predicted class', yaxis_title='Real class')
  ```

* Step 7. Show the prediction metrics using the plot\_heatmap function.

  ```python
  plot_heatmap(df_res,title='Classification results', 
      xaxis_title='Predicted class', yaxis_title='Prediction metric')
  ```

## Final thoughts / recommendation

Use this at your own risk!

Created by: Alberto Manzini

