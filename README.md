# Search Engine Optimization

## Introduction
The goal of this project is to provide a solution (a classification model) for optimizing search engine relevance. We want our model to be able to identify relevant responses and minimize irrelevant responses that are predicted relevant.

## Data
- 80046 observations (responses)
- 12 features, 1 binary target

## Workflow
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/flowchart.png" width="80%">

## EDA
- A [Tableau dashboard](https://public.tableau.com/profile/qi.feng1229#!/vizhome/SearchEngineOptimization_15900124420700/Dashboard1?publish=yes) was also created for interactive visualization.
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/seo_dashboard.png" width="80%">

## Preprocessing
- Scaling numerical features: RobustScaler
- Dealing with class imbalance: Class weight
- Preprocessing boosts validation ROC-AUC by 24.5%

## Models
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/2.3.1.png" width="80%">
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/2.3.2.png" width="80%">

- We can see that the ensemble classifiers such as XGBoost and Random Forest have higher validation ROC-AUC. The validation scores for simple linear models such as logistic regression and linear SVM are close to those of the ensembles. 
- In terms of time complexity, Logistic Regression is the fastest among the untuned baseline classifiers. For the fine-tuned ensemble classifiers, XGBoost is way faster than Random Forest.
- XGBoost is the best model in terms of model performance.

## Test Performance
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/xgb_test.png" width="40%">

- First, the test ROC-AUC is 0.65, which is lower than the validation ROC-AUC. Such a result indicates that our model might still be overfitting. The decrease in test score can also be attributed to the extent that the unseen test data is different from the training data. When splitting data, I set the test proportion to be 0.2 so that 20% of the dataset will go into the test set. If I set a lower proportion, chances are that the model can be trained on a larger training set and can generalize better.
- We can see that our model has higher precision, recall and F-1 for the label 0, which are the irrelevant responses, meaning that it is better at identifying irrelevant responses than relevant ones. For the relevant responses, precision is slightly higher than recall. Such a result is desired when we would like to have less False Positives in trade off to have more False Negatives, so that predicting an irrelevant response as relevant is more costly than predicting a relevant response as irrelevant. In a word, our search engine will not return a bunch of irrelevant responses, at the expense of ruling out some relevant responses.

## Feature Importance
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/xgb_feature_imp.png" width="65%">
- The importance of each feature is returned by the model based on how useful or valuable each feature was in the construction of the boosted decision trees The more a feature is used to make key decisions with decision trees, the higher its relative importance.

- We can see that ‘Sig2’ is the most important feature, followed by sig6, query_length, sig1, is_homepage and sig8. Such a result aligns with our initial exploration of the correlations between the target and the features.

## Business Insights
- If we shift the gear towards the business objective to maximize the profit, we will want to cut the operating cost related to the model, which should be proportional to the total computational resources for model fitting and prediction. 
- **Recommendation:** a logistic regression model with features identified as important by our best XGBoost model. This model is fine-tuned with Grid Search instead of random search because it only has a small number of parameter setting. As we observed before, logistic regression is computationally efficient. With four features dropped, it is as fast as 2.5 seconds, compared to 125 seconds of XGBoost. The test ROC-AUC is 0.63, which is comparable to XGBoost.
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/logit_comp.png" width="65%">

- If we look at the test performance of the model, we can see that our model still has higher precision, recall and F-1 for the irrelevant responses, so it is better at identifying irrelevant responses than relevant ones. But for the relevant responses, recall is now slightly higher than precision. This is more desirable if we decide the search engine should be more concerned about false negatives and be more reluctant to rule out relevant responses. With a higher recall, we can ensure that our search engine is capturing relevant responses, when the cost of missing a relevant response is higher than the cost of including an irrelevant response.
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/logit_test.png" width="40%">

### Actionable Insights
<img src="https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/img/pr.png" width="60%">

- First, precision is a measure of result relevancy. What’s the downside of erroneously identifying a response as likely to be relevant (false positive)? A low precision (large false positive) can undermine the search engine’s ability to filter out irrelevant responses, and could potentially elevate churn rate if users are unsatisfied with a bunch of irrelevant responses.
- Second, recall is a measure of how many truly relevant results are returned. What’s the downside of erroneously identifying a response as likely to be irrelevant (false negative)? A low recall (large false negative) will result in neglect of responses that are actually relevant. Again, a search engine with low recall can increase churn rate if the users are not able to find what they are searching for.
- Lastly, the relevance rate is a measure of how many responses can be returned back to the users as relevant overall. This depends on the cost of adding a response to the search result and possibly the capacity of the search engine interface.
- In a word, a search engine with high recall but low precision returns many results, but most of the results are irrelevant. A search engine with high precision but low recall is just the opposite, returning very few results, but most of the results are relevant. An ideal search engine should have high precision and high recall so that it returns many results and most of the results are relevant. But if we cannot avoid the trade-off, precision should be valued more over recall. 

- This plot shows precision, recall and the relevance rate as a proportion of total number of cases for each threshold selected for the classifier. A threshold is the decision boundary for classification. Remember that our predictions of the logistic regression are probabilities.
- If we choose a default threshold of 0.5, the model will classify a response as relevant if its predicted probability is larger than 0.5. - Now if we fix this value on the x-axis, we can see that: 
  - A relevant response is taken in about 45% of the cases.
  - Precision is about 60%, meaning that 60% of the responses that are predicted as relevant will actually be relevant. 
  - Recall is about 60%, which implies that about 40% of the relevant responses will not be returned as relevant. 
- I want to make it clear that the optimal choice of the threshold ultimately depends on the physical and financial constraints of the business. For instance, if the company wants to return relevant responses in about 75% of the cases, a threshold of about 0.4 will need to be chosen, in which case: 
  - Precision is about 50%, meaning that 50% of the responses that are predicted as relevant will actually be relevant. 
  - Recall is about 80%, which implies that about 20% of the relevant responses will not be returned as relevant. 

## Beyound This Point
- Several improvements can be made
  - Analysis of the optimal threshold
  - Stratified cross-validation for more representative data
  - Fixing overfitting issues in XGBoost: e.g., early stopping
  - Advanced models such as Light GBM
- For a detailed walk through of the project with methodology further explained, refer to [Search Engine Relevance Optimization Visual Walkthrough](https://github.com/lullaby1024/Search_Engine_Optimization/blob/master/Search%20Engine%20Relevance%20Optimization%20Visual%20Walkthrough.pdf).


