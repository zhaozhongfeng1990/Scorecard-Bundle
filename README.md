# ScorecardBundle

The one package you need for Scorecard modeling in Python.

**ScorecardBundle is a Python toolkit for Scorecard modeling of binary targets**. The transformer and model classes in ScorecardBundle comply with the fit-transform-predict convention in Scikit-learn.

There is a three-stage plan for ScorecardBundle:

- Stage 1: Replicate all functions of convectional Scorecard modeling, including Chi-Merge discretization, WOE transformation, IV calculation, Colinearity evaluation, Logistic regression training and Scorecard transformation.
- Stage 2: Automate the Scorecard modeling process, including:
  - Design algorithms to evaluate explainability (e.g. mean reversion)
  - Automatically select proper discretization methods for different features
  - 

<img src=".\pics\framework.png">

## Quick Start

First download the python scripts "ChiMerge", "WOE", "ScoreCard",  "ModelEvaluation",  "Score2Rating".

~~~python
import pickle
import sys
sys.path.append(r'D:\BaiduNetdiskDownload\0.codes\Algorithm') #put codes in this location
import ChiMerge as cm
import ScoreCard as sc
import WOE
import ModelEvaluation as me
import Score2Rating as rating

#load test data
path=r'D:\\Work\\data\\'

pkl_file = open(path+'testData.pkl', 'rb')
df, y_train_binary = pickle.load(pkl_file)
pkl_file.close()


# chi2
trans_chi2 = cm.ChiMerge(max_intervals=10,initial_intervals=100)
result1 = trans_chi2.fit_transform(df, y_train_binary)
#trans_chi2.A_dict

# woe
trans_woe = WOE.WOE(max_features=20)
result2 = trans_woe.fit_transform(result1, y_train_binary)

# scorecard 
scorecard = sc.ScoreCard(woe_transformer=trans_woe, decimal=0, PDO=-10, basePoints=60)
scorecard.fit(result2, y_train_binary)
scored_result = scorecard.predict(df)

# evaluate all
output_path = r'D:\\Work\\jupyter\\'
me.plot_all(y_train_binary.values, scored_result['TotalScore'].values, output_path=output_path)

# evaluate: if prefer to plot individually
me.ks(y_train_binary.values, scored_result['TotalScore'].values, output_path=output_path) 
me.roc(y_train_binary.values, scored_result['TotalScore'].values, output_path=output_path) 
me.precision_recall(y_train_binary.values, scored_result['TotalScore'].values, output_path=output_path) 
me.lift_curve(y_train_binary.values, scored_result['TotalScore'].values, output_path=output_path)
~~~

## Update Log

### Updates in v0.4

- ChiMerge：
  - When the distribution of a feature is heavily unbalanced (e.g. most values are the same), pandas.qcut will crash. Thus we will switch to pandas.cut durng the above circumstances.
- ModelEvaluation:
  - Fixed a bug in lift curve. Now the codes can generalize better.

- Scorecard
  - Add predict_proba() function to return scores only
  - Modify predict() and predict_proba() so that they support numpy array as input.

### Updates in v0.3

- ChiMerge
  - Fix a bug in ChiMerge that caused errors when bining data with pandas.qcut. If there are too many decimals in the minimum value of the column (e.g. 10 decimals), this minimum value would become the left boundary of the smallest interval procuced by qcut. The problem is that all intervals produced by qcut  are open on the left and close on the right. This means the smallest interval will not contain this minimum value.  To fix this, just round the column with pands.Series.round() before applying qcut.
  - Add a parameter `min_intervals`. When we don't want any feature droppped due to lack of predictability, we can use this parameter to make it happen.
- Scorecard
  - When using scorecard the data ranges may exceed those encountered in training, thus now the lowest and highest boundaries for each feature is set to negative infinity and positive infinity respectively.
- ModelEvaluation
  - If this module is run in jupyter notebook, the charts it saved to local used to be blank. This bug is fixed.

### Updates in v0.2

- Fix errors in notes. E.g. the default criterion for corr should be 0.6 rather than 0.7;
- Add example of using sklearn.utils.class_weight;
- Make Sure most default parameters  are optimal for Suitability scorecard model; 