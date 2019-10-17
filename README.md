# Starbucks Simulated Marketing Project
Analysing Starbucks promotional data for Udacity Data Science capstone project.

### Table of Contents

1. [Installation](#installation)
2. [Project Motivation](#motivation)
3. [Data limiations](#data)
4. [Main Findings](#findings)

## Installation <a name="installation"></a>

This code uses Python 3 and the packages listed at the top of the notebook. 
These are pandas (0.23.3), numpy (1.12.1), matplotlib (2.1.0), seaborn (0.9.0), sklearn (0.19.1) and joblib (0.11).

Seaborn has to be at least version 0.9.0 to support the scatterplots.

## Project Motivation<a name="motivation"></a>

The purpose of this project was to complete the final assignment in a Udacity Data Science Nano-Degree.
Data was provided from Starbucks, which is simulated customer, marketing and transaction information.
I set out to find insights in this data that could increase revenue.

## Data limiations<a name="data"></a>

As this was simulated data, there were a few quirks in the data which you would not expect to see in live Starbucks data. These include:
* The distribution of ages look to be what you would see at a country population level, not at a customer level. For example There were more members that were over 80, than in the age group 18 to 25.
* While you would expect to see age correlated with income, in the data there are 3 jumps in the minimum values by age.

## Main Findings<a name="findings"></a>

* There were members where no demographic data was held against them. They showed to be less likely to view and complete an offer, and more likely to get a discount without being influenced by the promotion. We would therefore recommend that costly offers such as discounts or BOGO are not sent to these customers.
* Social media is the best channel to use, although if there is no extra cost multiple channels can be used. The one exception is younger lower income groups (potentially male and newer members as well) react adversely to mobile advertisements, so an A/B test could be carried out to see if there is a boost by not using that channel for them.




