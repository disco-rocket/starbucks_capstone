# Starbucks Simulated Marketing Project
Analysing Starbucks promotional data for Udacity Data Science capstone project.

### Table of Contents

1. [Installation](#installation)
2. [Project Overview](#overview)
3. [Problem Statement](#problem)
4. [Data Abnormalities](#data)
5. [Preprocessing](#pre)
6. [Metrics](#metrics)
7. [Model Choices](#choices)
8. [Conclusions](#conclusions)
9. [Improvements](#improvements)

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

Starbucks is one of the largest retailers in coffee in the world, with [89% of revenue coming from the 18 to 40 demographic](https://brandongaille.com/30-curious-starbucks-demographics/).

The purpose of this project was to complete the final assignment in a Udacity Data Science Nano-Degree.
Data was provided by Starbucks, which is simulated customer, marketing and transaction information.
I set out to find insights in this data that could increase revenue.

An article further discussing the analysis was published in [Medium](https://medium.com/@sheffseankiely/what-simulated-starbucks-data-tells-us-about-marketing-8d5097858cc2?sk=04b9d9b99967dfea2676f9673ffd7bdc):

Implicit consent to use the data was given for the nano-degree course, but there is no known permission to use the data beyond this use.
The data is simulated, so no real world conclusions should be drawn from either the results or data.

## Problem Statement<a name="problem"></a>
We want to find ways to maximise revenue by (1) focusing on customers who will respond to promotions, and (2) focusing on promotions that customers will best respond.
To do this we will focus on the promotions data, and append customer and outcome information to this base data. The effectiveness of each promotion can then be tracked.
The solution approach will be to
* Use clustering to seperate the customers into different populations to analyse
* Look at the propotion of completions, both where the offer was viewed or not, for each identified cluster. This will give an indication of what customer demographics may be the best focus of offers
* Create a model to analyse both demographic and offer data at the same time, to see what gives the most impact for completions
* Create seperate models on just the offer metrics for each of the clusters. This will give an indication of what is most important to different customer groups.

## Data Abnormalities<a name="data"></a>

There are 12.8% of the members where the demographic details were missing. These were null for gender and income, and the missing values for ages were encoded as 118.

As this was simulated data, there were a few quirks in the data which you would not expect to see in live Starbucks data. These include:
* The distribution of ages look to be what you would see at a country population level, not at a customer level. For example there were more members that were over 80, than in the age group 18 to 25.
* While you would expect to see age correlated with income, in the data there are 3 jumps in the minimum values by age, which doesn't look organic.

## Preprocessing<a name="pre"></a>

The field for the date of membership is an integer stored in yyyymmdd format. As dd will only go up to a maximum of 31, and mm will go up to a maximum of 12, it cannot be scaled in the normal way. In order to get around this a new field was created which was the number of days from the date that the first member was created. This date can then be scaled in the normal way.

The transcript data contains information on the offer received, offer viewed, offer completed and transactions in one table. The attributes against each event is stored in a single column called value. 
This contains a dictionary with keys of 'reward' and 'offer_id' for 'offer completed' events, 'offer id' (without the underscore) for  'offer received' and 'offer viewed', and finaly 'amount' for 'transaction' events.

The offer received data was extracted from the transcript data, and the offer viewed data merged on using the 'offer id' and 'person'.
This resulting dataframe has the time (in hours from the start of the first offer) of both the offer recieved and viewed, as well as the duration (in days) that the offer is considered valid.
These fields are used to restrict down the dataframe from where the viewing was after the offer was sent, but the viewing occured before the offer had expired. Finally this creates a binary flag for whether the offer was used or not.

This process was repeated to get a flag for the completed data.

The final flag was taken by combining the completed flag, with the viewed flag to get where the offer was both viewed and completed. It was also used to create another binary flag for where the offer was completed but not viewed.

### Complications
* The seaborn package that deals with scatter plots is only in version 0.9.0. I therefore had to add a bit at the start of the code to make sure the Conda environment had the latest version.

## Metrics<a name="metrics"></a>
There are two target that will be derived from the data.
1. Whether the promotion was followed up with a qualitifying transaction. This will allow us to maximise revenue by focusing on factors that maximise this rate.
2. Whether the promotion was followed up with a qualifying transactions, despite the promotion not being viewed. This will allow us to identify customer segments that would have made a purchase anyway at full price or without the promotion.

Clustering on the demographic data will be used to see if any of the clusteres are over or underrepresented in the above two populations.

Logistic regression will then be used on the combined demographic and offer data to get the coefficients to see what the major factors are for completeing an offer, and then the accuracy on this model will be used to verify that these coefficients adequatly explain the variance.

The accuracy will be calculated on a testing dataset, which is the last 20% of rows in the promotions dataset. These last 20% will be a better mix of promotions to new and existing customers, and will be more likely to generalise to live data.

This regression will then be repeated on the different demographic groups identified in the clustering to look for differences in the coefficients between the groups.

Finally the accuracy on other models will be looked at, to see if we can explain significantly more variance using techniques that can more accuratly classifiy on non-linear clusters.

Accuracy is chosen as the completion rates are around 37%, so it isn't too unbalanced to skew the results. Also false positives and false negatives are equally important in identifying who to focus resources on, so other metrics such as recall and precision are less important.

## Model choices<a name="choices"></a>
Transforming become_member_on to days_from_first_member
### Changing gender variable to binary type
There was male, female, then a small proportion of others. If we hot encoded it then it may cause issues with multi-colinearity, and because male is the biggest group, adding other to it shouldn't skew it too much.
### MinMaxScaler
MinMaxScaler was chosen because it will keep values between 0 and 1, which will match with the binary variables.
More accurate classifications could have been achieved using StandardScaler, but as we are looking for insights instead of high accuracy MinMaxScaler is good.
### imputation
Imputation was not used, because the completion rates were almost 4 times higher in the population with nulls than without nulls. They were therefore analysed seperatly instead.
### kmeans
Kmeans was chosen because it effectivly splits up the data into groups, even where the data does not have clear clusters (unlike algorthyms that are better with clearly distinct data like dbscan or MeanShift).
It is also very efficient (especially with MiniBatchKmeans), so it runs faster than algorythms like AffinityPropagation.
### logistic regression
Logistic regression was chosen because the coefficients it creates give insights around the scale of impact of each variable (they are the log of the odds).
It does well with data where the impact of the dimensions is roughly linear, but it will not do well if the data has dimensions that have pockets of impact (for example if ages 40 to 50 are more prone to purchasing than the other ages). To get around this, other models were used to see how much the accuracy could be improved - big improvements in accuracy could be indicative of such pockets.
The accuracy was 73.4% (+/- 0.5%) on the training dataset (using cross validation), and 73.2% on the testing dataset
### Adaboost Classifier
This was chosen because it is relatively quick to run (compared with more complex models such as SVC), but it will handle pockets of insights better than logistic regression.
The accuracy on the test data was 75.38%
Training accuracy of 75.6% (+/- 0.5), and a test accuracy of 75.4%
### Random Forest Classifier
This is similar to Adaboost (it uses the same base classifier), but is a slightly different algorythm.
The accuracy on the training data was 75.5% (+/- 1.1), and a test accuracy of 75.4%.
### Refinements
Adaboost had less variability and a higher training score, so that was then optimised using GridSearchCV. The parameters looked at were learning rate (0.1, 0.5, 1, 5, 10), n_estimators (10, 50, 100) and 3 different base estimators (all decision trees, but with max depths of 1, 2 and 3). The best model was where the learning rate was 0.1 and 100 estimators with a maximum depth of 3. This boosted the accuracy on the training data of 77.00% (+/- 0.5%), and an accuracy on the test data of 77.28%.

This refinement tells us that there may be pockets of insights which logistic regression didn't highlight, but the increase isn't big enough to invalidate the insights gleaned from the coefficients.

## Conclusions<a name="conclusions"></a>

* The first step we went through was clustering the demographic information, which resulted in 6 clusters to work from. A 7th cluster was added where there was no demographic data present.
* Where there as no data present (segment 7), they had lower representations in the completed and viewed population, ann higher representation in the completed and not viewed population. We would therefore recommend that costly offers such as discounts or BOGO are not sent to these customers.
* When using logistic regression on both the demographic and offer data, by far the biggest impact was the type of offer (holding all other factors constant, an informational offer would decrease the chance of completeing by 99.7%, compared with a discount which would increase it by around 7 times. The biggest demographic impact was the income, and the longer a customer had been a member the more likely they are to complete.
* Most of the clusters got similar regression values, with the main exception of group 4 (younger male, lower income) who reacted negatively instead of positively to mobile advertisements.

Ultimatly this was on simulated data, which did show in some of the scatter plots. It would be more interesting to try this on real world data, which would also add more depth to the reccomendations. 
It was however interesting how the different models generalised well to the test data, even though the test data contained recent data with (in theory) a different mix of customers.

## Improvements<a name="improvements"></a>
* Fairly simple clustering and supervised learning algorythms were used. We could try more complex and resource intensive models such as SVM for supervised learning, and dbscan for clustering. This could have provided more accurate clusters and models, but would have taken more time to run (especially when optimising using gridsearch).
* For the ensemble methods, we could dive into the resulting base esimators to try to find more granular insights such as age pockets that react differently. This contrasts with our method which will only give insights on the impact of the size of variables instead of groupings.
* A/B tests could be conducted to confirm some of our conclusions. This would be good because the current design, where multiple features are correlated risks making the models less accurate due to multicollinearity.








