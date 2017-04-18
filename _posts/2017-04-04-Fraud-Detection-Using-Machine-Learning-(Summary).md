---
layout: single
header:
  overlay_image: /assets/Fraud-Detection-Using-Machine-Learning/banner.jpg
  overlay_filter: 0.5
  caption: "Photo credit: [**Impawards**](http://www.impawards.com/)"
  teaser: https://blog.ordoro.com/wp-content/uploads/2012/07/fraud.jpg
title:  "Fraud Detection Using Machine Learning (Summary)"
excerpt: "Identify which employees are more likely to have committed fraud by applying machine learning to financial and email data."
date:   2017-04-04 15:26:53 +0300
tags:
- Python
- Scikit-learn
- Machine learning
- Natural language processing
- Feature selection
- Verifying machine learning performance
---

{% include toc %}

# Introduction

In 2000, Enron was one of the largest companies in the United States. By 2002, it had collapsed into bankruptcy due to widespread corporate fraud. In the resulting Federal investigation, a significant amount of typically confidential information entered into the public record, including tens of thousands of emails and detailed financial data for top executives.
These data have been combined with a hand-generated list of persons of interest in the fraud case, which means individuals who were indicted, reached a settlement or plea deal with the government, or testified in exchange for prosecution immunity. These data have created a dataset of 21 features for 146 employees.

The scope of the project is the creation of an algorithm with the ability to identify Enron Employees who may have committed fraud. To achieve this goal, Exploratory Data Analysis and Machine Learning are deployed to clear the dataset from outliers, identify new parameters and classify the employees as potential Persons of Interest.  

(**Note**: *This is just an overview of the project. You may find the full analysis on this [Notebook](https://github.com/YannisPap/Identify-Fraud-using-Machine-Learning/blob/master/Identify%20Fraud%20Using%20Machine%20Learning%20.ipynb).*)

# Data Exploration

The features included in the dataset can be divided in three categories, Payment Features, Stock Features and Email Features. Bellow you may find the full feature list with  brief definition of each one.

**Payment Features**

| Payments            | Definitions of Category Groupings|
|:--------------------|:---------------------------------|
| **Salary**              | Reflects items such as base salary, executive cash allowances, and benefits payments.|
| **Bonus**               | Reflects annual cash incentives paid based upon company performance. Also may include other retention payments.|
| **Long Term Incentive** | Reflects long-term incentive cash payments from various long-term incentive programs designed to tie executive compensation to long-term success as measuredagainst key performance drivers and business objectives over a multi-year period, generally 3 to 5 years.|
| **Deferred Income**     | Reflects voluntary executive deferrals of salary, annual cash incentives, and long-term cash incentives as well as cash fees deferred by non-employee directorsunder a deferred compensation arrangement. May also reflect deferrals under a stock option or phantom stock unit in lieu of cash arrangement.|
|**Deferral Payments**   | Reflects distributions from a deferred compensation arrangement due to termination of employment or due to in-service withdrawals as per plan provisions.|
| **Loan Advances**       | Reflects total amount of loan advances, excluding repayments, provided by the Debtor in return for a promise of repayment. In certain instances, the terms of thepromissory notes allow for the option to repay with stock of the company.|
| **Other**               | Reflects items such as payments for severence, consulting services, relocation costs, tax advances and allowances for employees on international assignment (i.e.housing allowances, cost of living allowances, payments under Enron’s Tax Equalization Program, etc.). May also include payments provided with respect toemployment agreements, as well as imputed income amounts for such things as use of corporate aircraft. |
| **Expenses**            | Reflects reimbursements of business expenses. May include fees paid for consulting services.|
| **Director Fees**       | Reflects cash payments and/or value of stock grants made in lieu of cash payments to non-employee directors.|
| **Total Payments**      | Sum of the above values|

***

**Stock Features**

| Stock Value              | Definitions of Category Groupings|
|:-------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Exercised Stock Options**  | Reflects amounts from exercised stock options which equal the market value in excess of the exercise price on the date the options were exercised either throughcashless (same-day sale), stock swap or cash exercises. The reflected gain may differ from that realized by the insider due to fluctuations in the market price andthe timing of any subsequent sale of the securities. |
| **Restricted Stock**         | Reflects the gross fair market value of shares and accrued dividends (and/or phantom units and dividend equivalents) on the date of release due to lapse of vestingperiods, regardless of whether deferred.|
| **Restricted StockDeferred** | Reflects value of restricted stock voluntarily deferred prior to release under a deferred compensation arrangement.|
| **Total Stock Value**        | Sum of the above values |

***

**email features**

| Variable                      | Definition                                                                    |
|:------------------------------|:------------------------------------------------------------------------------|
| ***to messages***             | Total number of emails received (person's inbox)                              |
| ***email address***           | Email address of the person                                                   |
| ***from poi to this person*** | Number of emails received by POIs                                             |
| ***from messages***           | Total number of emails sent by this person                                    |
| ***from this person to poi*** | Number of emails sent by this person to a POI.                                |
| ***shared receipt with poi*** | Number of emails addressed by someone else to a POI where this person was CC. |

The dataset was quite sparse with some of the above features having too few values.  
The missing values were actually "0" and they imputed accordingly.

|Feature                  | Observations #|
|:------------------------|--------------:|
|loan_advances            |      4        |
|director_fees            |     17        |
|restricted_stock_deferred|     18        |
|deferral_payments        |     39        |
|deferred_income          |     49        |
|long_term_incentive      |     66        |
|bonus                    |     82        |
|to_messages              |     86        |
|shared_receipt_with_poi  |     86        |
|from_this_person_to_poi  |     86        |
|from_poi_to_this_person  |     86        |
|from_messages            |     86        |
|other                    |     93        |
|salary                   |     95        |
|expenses                 |     95        |
|exercised_stock_options  |    102        |
|restricted_stock         |    110        |
|email_address            |    111        |
|total_payments           |    125        |
|total_stock_value        |    126        |

These data have created an initial dataset of 20 features for 146 employees.

During the process the following outliers were revealed probably as a result of the extraction from the [Payments Schedule]({{ site.url }}/assets/2017-04-04-Fraud-Detection-Using-Machine-Learning/enron61702insiderpay.pdf):

* a datapoint named '*TOTAL*' matching the totals row from the Schedule
* a datapoint named "*THE TRAVEL AGENCY IN THE PARK*" which was not representing an employee
* an employee (*LOCKHART EUGENE E*) with no values across all features.
* two employees (*BELFER ROBERT* and *BHATNAGAR SANJAY*) with some values transposed between columns  

The first three outliers were removed and the rest were corrected. There were also a lot of employees with extreme values (Tukey's Method wit 3 x IQR revealed 93 employees with at least one extreme value) but because of the nature of the problem and the size of the dataset, I decided to keep them.  
These transformations brought the dataset to its final size, 143 employees x 19 features (I removed 'email_address' from features).
 
Finally, the dataset was very imbalanced between the two classes with only 18 POIs in the 143 employees. For this reason instead of using the Accuracy to measure the effectiveness, I used the F1 metric which takes into account both Precision and Recall.

# Feature Selection/Engineering

There are some cases where the value of a variable might be less important than its proportion to a related aggregated value. As an example from the current dataset, a bonus of 100,000 is less informative than a bonus 3 times the salary, or "500 sent email to POIs" is far less informative than "half of the sent emails have been sent to POIs".
For this reason and since all the features were related to an aggregated value, I created the proportions of all the features to their respective aggregated value. These new features added to the dataset and the 'enchanced' dataset evaluated with the ```SelectPercentile(percentile=100)```.  
![features_importance]({{ site.url }}/assets/2017-04-04-Fraud-Detection-Using-Machine-Learning/features_importance.png)  

The result showed that the following proportions were more significant than the related original feature: 
* *Long Term Incentive* to *Total Payments*
* *Restricted Stock Deferred* to *Total Stock Value*
* *From This Person to POI* to *FROM Messages*  

They added to the dataset and in the same time removed the original features to avoid any bias towards these features.  

![features_importance]({{ site.url }}/assets/2017-04-04-Fraud-Detection-Using-Machine-Learning/features_importance2.png)
The used classifier is not based on recursive partitioning, so scaling was required. Since the dataset was quite sparse, ```MaxAbsScaler()``` was selected to preserve the sparseness structure in the data. The accuracy of the model (using LinearSVC and F1 metric for the evaluation) comparing to the number of features after the above procedure were:  

![feature_reduction]({{ site.url }}/assets/2017-04-04-Fraud-Detection-Using-Machine-Learning/feature_reduction.png)


Afterword, I evaluated several classifiers both with Univariate Feature Selection and Primary Component Analysis using GridSearchCV and I ended up with PCA with 2 principal components.

# Algorithm Selection

The most appropriate algorithm for the specific case was **Nearest Centroid**. Bellow you may find all the evaluated algorithms and their performance.

|           Category           |        Algorithm       |   Accuracy  |  Precision  |    Recall   |      F1     |      F2     |
|:----------------------------:|:----------------------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
|    Support Vector Machine    |        LinearSVC       |   0.76007   |   0.24481   |   0.38350   |   0.29885   |   0.34447   |
|    Support Vector Machine    |           SVC          | **0.86927** | **0.88235** |   0.02250   |   0.04388   |   0.02795   |
| Nearest Neighbors            | KNeighborsClassifier   | 0.85193     | 0.43233     | 0.35300     | 0.38866     | 0.36645     |
| **Nearest Neighbors**        | **NearestCentroid**    | 0.73833     | 0.30975     | **0.78350** | **0.44397** | **0.59997** |
| Ensemble Methods (Averaging) | RandomForestClassifier | 0.83293     | 0.36427     | 0.33950     | 0.35145     | 0.34418     |
| Ensemble Methods (Boosting)  | AdaBoostClassifier     | 0.84893     | 0.40404     | 0.28000     | 0.33077     | 0.29832     |


As can be seen in the table, Support Vector Classifier performed better in Accuracy and Precision and Nearest Centroid in Recall and the F scores. I ended up using Nearest Centroid because I wanted a more balanced behavior, otherwise a high score may be misleading if it is combined with poor score in the other categories. This can be demonstrated graphically.  

|SVC                    |Nearest Centroid                                 |
|:---------------------:|:-----------------------------------------------:|
|![SVC]({{ site.url }}/assets/2017-04-04-Fraud-Detection-Using-Machine-Learning/svc.png)|![Nearest Centroid]({{ site.url }}/assets/2017-04-04-Fraud-Detection-Using-Machine-Learning/nearest_centroid.png)|

It is clear that the extremely high (comparing to the rest) Precision of SVC is because it evaluates very "conservative" the datapoints. It makes two right picks but it can only spot 2 out of 18 POIs.  
On the other hand, Nearest Centroid has some false positives but in general can better distinct POIs from non-POIs.

# Hyperparameters Optimization

For parameter optimization I used Exhaustive Grid Search to conclude to the following parameters:

|      Process      |    Algorithm    |     Parameter    |      Evaluated Values      |  Selected Value  |
|:-----------------:|:---------------:|:----------------:|:--------------------------:|:----------------:|
|      Scaling      |  MaxAbsScaller  |       copy       |      True                  | True             |
| Feature Selection |       PCA       |   n_components   |        [2, 3, 4, 5]        |         2        |
|   Classification  | NearestCentroid |      metric      | ["euclidean", "manhattan"] |    "manhattan"   |
|                   |                 | shrink_threshold |     [None, 0.1, 1, 10]     |       None       |

***
(*Note: Additional Scaling and Feature Selection methods were evaluated, but they didn't performed as well as the above.*)
