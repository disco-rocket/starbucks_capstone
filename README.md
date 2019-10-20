# Starbucks Simulated Marketing Project
Analysing Starbucks promotional data for Udacity Data Science capstone project.

### Table of Contents

1. [Installation](#installation)
2. [Project Overview](#overview)
3. [Problem Statement](#problem)
4. [Metrics](#metrics)
5. [Data Limiations](#data)
6. [Model Choices](#choices)
7. [Conclusions](#conclusions)

### File structure

    ├── data                    
    │   ├── portfolio.json  # containing offer ids and meta data about each offer (duration, type, etc.)
    │   ├── profile.json    # demographic data for each customer
    │   └── transcript.json # records for transactions, offers received, offers viewed, and offers completed
    ├── pictures                    # Outputs used in the medium article
    │   ├── correlation_matrix.png  # CSV containing the categories for disaster messages  
    │   ├── demographic_groups.png  # CSV containing the categories for disaster messages
    │   ├── elbow_graph.png         # CSV containing the actual disaster messages
    │   ├── groups_by_outcomes.png  # CSV containing the categories for disaster messages
    │   ├── groups_table.png        # CSV containing the actual disaster messages
    │   └── missing_comparisons.png # Code to launch the webapp. Plotly graphs are also set up in this code.
    ├── README.md                         # README
    └── Starbucks_Capstone_notebook.ipynb # Folder containing screenshots

## Installation <a name="installation"></a>

This code uses Python 3 and the packages listed at the top of the notebook. 
These are pandas (0.23.3), numpy (1.12.1), matplotlib (2.1.0), seaborn (0.9.0), sklearn (0.19.1) and joblib (0.11).

Seaborn has to be at least version 0.9.0 to support the scatterplots.

## Project Overview<a name="overview"></a>

The purpose of this project was to complete the final assignment in a Udacity Data Science Nano-Degree.
Data was provided by Starbucks, which is simulated customer, marketing and transaction information.
I set out to find insights in this data that could increase revenue.

An article further discussing the analysis was published in [Medium](https://medium.com/@sheffseankiely/what-simulated-starbucks-data-tells-us-about-marketing-8d5097858cc2?sk=04b9d9b99967dfea2676f9673ffd7bdc):

Implicit consent to use the data was given for the nano-degree course, but there is no known permission to use the data beyond this use.
The data is simulated, so no real world conclusions should be drawn from either the results or data.

### Problem Statement<a name="problem"></a>
We want to find ways to maximise revenue by (1) focusing on customers who will respond to promotions, and (2) focusing on promotions that customers will best respond.
To do this we will focus on the promotions data, and append customer and outcome information to this base data. The effectiveness of each promotion can then be tracked.

### Metrics<a name="metrics"></a>
There are two target that will be derived from the data.
1. Whether the promotion was followed up with a qualitifying transaction. This will allow us to maximise revenue by focusing on factors that maximise this rate.
2. Whether the promotion was followed up with a qualifying transactions, despite the promotion not being viewed. This will allow us to identify customer segments that would have made a purchase anyway at full price or without the promotion.

Clustering on the demographic data will be used to see if any of the clusteres are over or underrepresented in the above two populations.

Logistic regression will then be used on the combined demographic and offer data to get the coefficients to see what the major factors are for completeing an offer, and then the accuracy on this model will be used to verify that these coefficients adequatly explain the variance.

The accuracy will be calculated on a testing dataset, which is the last 20% of rows in the promotions dataset. These last 20% will be a better mix of promotions to new and existing customers, and will be more likely to generalise to live data.

This regression will then be repeated on the different demographic groups identified in the clustering to look for differences in the coefficients between the groups.

Finally the accuracy on other models will be looked at, to see if we can explain significantly more variance using techniques that can more accuratly classifiy on non-linear clusters.

Accuracy is chosen as the completion rates are around 37%, so it isn't too unbalanced to skew the results. Also false positives and false negatives are equally important in identifying who to focus resources on, so other metrics such as recall and precision are less important.

## Data limiations<a name="data"></a>

As this was simulated data, there were a few quirks in the data which you would not expect to see in live Starbucks data. These include:
* The distribution of ages look to be what you would see at a country population level, not at a customer level. For example there were more members that were over 80, than in the age group 18 to 25.
* While you would expect to see age correlated with income, in the data there are 3 jumps in the minimum values by age, which doesn't look organic.

## Model choices<a name="choices"></a>
Transforming become_member_on to days_from_first_member
#### Changing gender variable to binary type
There was male, female, then a small proportion of others. If we hot encoded it then it may cause issues with multi-colinearity, and because male is the biggest group, adding other to it shouldn't skew it too much.
#### MinMaxScaler
MinMaxScaler was chosen because it will keep values between 0 and 1, which will match with the binary variables.
More accurate classifications could have been achieved using StandardScaler, but as we are looking for insights instead of high accuracy MinMaxScaler is good.
#### imputation
Imputation was not used, because the completion rates were almost 4 times higher in the population with nulls than without nulls. They were therefore analysed seperatly instead.
#### kmeans
Kmeans was chosen because it effectivly splits up the data into groups, even where the data does not have clear clusters (unlike algorthyms that are better with clearly distinct data like dbscan or MeanShift).
It is also very efficient (especially with MiniBatchKmeans), so it runs faster than algorythms like AffinityPropagation.
#### logistic regression
Logistic regression was chosen because the coefficients it creates give insights around the scale of impact of each variable (they are the log of the odds).
It does well with data where the impact of the dimensions is roughly linear, but it will not do well if the data has dimensions that have pockets of impact (for example if ages 40 to 50 are more prone to purchasing than the other ages). To get around this, other models were used to see how much the accuracy could be improved - big improvements in accuracy could be indicative of such pockets.
The accuracy on the test data was 73.2%
#### Adaboost Classifier
This was chosen because it is relatively quick to run (compared with more complex models such as SVC), but it will handle pockets of insights better than logistic regression.
The accuracy on the test data was 75.38%
#### Random Forest Classifier
This is similar to Adaboost (it uses the same base classifier), but is a slightly different algorythm.
The accuracy on the test data is almost the same, but slightly below Adaboost at 75.35%
#### Tuning Adaboost.
Gridsearch was used to maximise the accuracy on the training dataset, which returned an accuracy on the test data of 77.28%. Although this is an improvement, its not big enough of an improvement over logistic regression to suggest that the insights from the coeeficient aren't correct. 
Further work could have been done to look for over-fitting by comparing the accuracy of the training and test data, but since we are just looking to validate the logistic regression approach, this won't add anything further to the analysis.
## Conclusions<a name="conclusions"></a>
The main findings were:
* There were members where no demographic data was held against them. They showed to be less likely to view and complete an offer, and more likely to get a discount without being influenced by the promotion. We would therefore recommend that costly offers such as discounts or BOGO are not sent to these customers.
* Social media is the best channel to use, although if there is no extra cost multiple channels can be used. The one exception is younger lower income groups (potentially male and newer members as well) react adversely to mobile advertisements, so an A/B test could be carried out to see if there is a boost by not using that channel for them.

Without having information on the cost of the promotions to Starbucks it was not possible to look to maximise profitability.
Further work could be done if that information were to be provided







