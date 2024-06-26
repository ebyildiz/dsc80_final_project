# Predicting Demand Loss in Major Power Outages in Different States
*by Elif Yildiz*

## Introduction

In an increasingly electricity-dependent world, understanding the factors contributing to major power outages is vital for minimizing economic and societal disruptions. This project analyzes a comprehensive dataset documenting significant power outages across the United States from January 2000 to July 2016 across different states. Defined by the Department of Energy, these major outages impacted at least 50,000 customers or resulted in an unplanned firm load loss of at least 300 MW.

The dataset is robust, encompassing not only the primary details of the outages, such as their geographical locations, dates, and times, but also data on regional climatic conditions, land-use characteristics, electricity consumption patterns, and economic attributes of the affected states. This multifaceted approach allows for a nuanced analysis of the factors influencing power outages and provides a fertile ground for developing predictive models to estimate demand loss based on various predictors. 

The dataset is available at Purdue University’s Laboratory for Advancing Sustainable Critical Infrastructure, [link to dataset](https://engineering.purdue.edu/LASCI/research-data/outages)

## Data Cleaning and Exploratory Data Analysis

I will start my analysis by cleaning the data, and limiting it to only the columns I need since it contains 57 columns. Some of the columns like 'u.s._state' and 'postal.code' contain almost the same information, which can lead to multicollinearity. The same thing happens for the anomaly levels and climate category as they are highly correlated. Because of this, I will only be using one of each. For the sake of simplicity, I converted all the columns to lower case.

I have also excluded columns that won't be available at the time of predicting the demand loss such as outage duration or restoration date.

Let's take a look at the columns:

| Column | Description |
| ----------- | ----------- |
| `year` | Indicates the year when the outage event occurred |
| `month` | Indicates the month when the outage event occurred |
| `u.s._state` | State the outage occured in |
| `nerc.region` | The North American Electric Reliability Corporation (NERC) regions involved in the outage event |
| `climate.category` | Climate category as warm, cold or normal and episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI) |
| `outage.start.date` | Date of when the outage has started |
| `outage.start.time` | Time of when the outage has started |
| `cause.category` | The general cause of the outage |
| `cause.category.detail` | The detailed cause of the outage |
| `demand.loss.mw` | Amount of peak demand lost during an outage event (in Megawatt) [but in many cases, total demand is reported] This is also the columns we want to predict |
| `customers.affected` | Number of customers affected by the power outage event |
| `res.sales`	| Electricity consumption in the residential sector (megawatt-hour) |
| `com.sales`	| Electricity consumption in the commercial sector (megawatt-hour) |
| `ind.sales`	| Electricity consumption in the industrial sector (megawatt-hour) |
| `total.sales`| Total electricity consumption in the U.S. state (megawatt-hour) |
| `population` | Population in the U.S. state in a year |
| `popden_urban` | Population density of the urban areas (persons per square mile) |
| `popden_rural` | Population density of the rural areas (persons per square mile) |
| `popden_uc` | Population density of the urban clusters (persons per square mile) |


By examining historical trends and patterns, the dataset facilitates a deeper understanding of the dynamics of power outages. It enables the identification and assessment of key risk factors associated with sustained power outages, thus offering valuable insights for policymakers, utility companies, and researchers. The ultimate goal of this project is to enhance the predictive capabilities concerning demand loss due to different causes of outages, thereby contributing to more resilient and responsive power systems.

### Data Cleaning Process

  First, I start by dropping columns that won't be necessary for my analysis. These include columns such as land area which can make my model for predicting demand loss more complicated as they are probably not as highly correlated as other columns such as cause category or state. I also wanted to get rid of columns that will not be available at the time of prediction for demand loss. These include information about how long the outage was or when it was restored since we want to predict demand loss as soon as the outage happens. Some other columns I dropped were highly correlated columns such as anomaly level and climate category to avoid multicollinearity. This was a tricky one since anomaly level is a numeric continuous data while climate category is categorical. I have come to the conclusion that it might be more beneficial to use climate category for the case of my analysis since it is more simple and anomaly level might increase the complexity of the model a little bit to high, causing too much variation.

  Additionally, I dropped columns that I don't think would be as relevant to my analysis such as the hurricane type which also had a lot of missing values since most of the outages are not caused by hurricanes.
  
  After dropping these columns, I ended up with the ones I show above which are : [`year`, `month`, `u.s._state`, `nerc.region`, `climate.category`, `outage.start.date`, `outage.start.time`, `cause.category`, `cause.category.detail`,  `demand.loss.mw`, `customers.affected`, `res.sales`, `com.sales`, `ind.sales`, `total.sales`, `population`, `popden_urban`, `popden_uc`, `popden_rural`]

  I also wanted to combine the columns outage start date and time as one series with dtype timestamp object in `outage.start` column. I dropped the other columns since I will not need them anymore.

  As I was looking at the dataset, I have realized that demand loss and customers affected had a lot of 0 values. Since this cannot be possible, I decided to change them with nan values. After that, I also dropped the rows where demand loss had missing values because it is the column I am trying to predict and missing values can mess with my model.

  Here is a small sample of the dataset to get an idea of what we are dealing with:

|   OBS |   year |   month | u.s._state   | nerc.region   | climate.category   | outage.start.date            | outage.start.time   | cause.category                | cause.category.detail   |   demand.loss.mw |   customers.affected |        res.sales |        com.sales |        ind.sales |      total.sales |   population |   popden_urban |   popden_uc |   popden_rural |
|------:|-------:|--------:|:-------------|:--------------|:-------------------|:-----------------------------|:--------------------|:------------------------------|:------------------------|-----------------:|---------------------:|-----------------:|-----------------:|-----------------:|-----------------:|-------------:|---------------:|------------:|---------------:|
|  1065 |   2004 |       9 | Florida      | FRCC          | warm               | Sunday, September 26, 2004   | 2:00:00 AM          | severe weather                | hurricanes              |             1250 |               285300 |      1.05573e+07 |      7.63206e+06 |      1.62635e+06 |      1.9824e+07  |     17415318 |         2315.2 |      1230.3 |           35.9 |
|  1382 |   2015 |       2 | Georgia      | SERC          | warm               | Monday, February 16, 2015    | 9:41:00 PM          | severe weather                | winter                  |              620 |               186035 |      5.2295e+06  |      3.5561e+06  |      2.48048e+06 |      1.12806e+07 |     10214860 |         1516   |      1112.2 |           45.8 |
|  1138 |   2016 |       4 | California   | WECC          | warm               | Saturday, April 2, 2016      | 11:08:00 AM         | system operability disruption | uncontrolled loss       |              360 |                  nan |      5.91565e+06 |      8.99193e+06 |      3.96474e+06 |      1.89413e+07 |     39296476 |         4303.7 |      2124.1 |           12.7 |
|  1053 |   2005 |       9 | Florida      | FRCC          | normal             | Thursday, September 22, 2005 | 12:00:00 PM         | severe weather                | nan                     |                0 |                    0 |      1.24811e+07 |      8.65359e+06 |      1.66117e+06 |      2.28044e+07 |     17842038 |         2315.2 |      1230.3 |           35.9 |
|   155 |   2006 |       7 | Michigan     | RFC           | normal             | Sunday, July 16, 2006        | 2:00:00 PM          | severe weather                | thunderstorm            |              150 |               315000 |      3.95857e+06 |      3.71942e+06 |      2.96104e+06 |      1.06392e+07 |     10036081 |         2034.1 |      1390.4 |           47.5 |
|   511 |   2011 |       4 | Arizona      | WECC          | cold               | Thursday, April 28, 2011     | 4:09:00 PM          | equipment failure             | nan                     |              960 |                  nan |      1.96443e+06 |      2.2737e+06  | 988825           |      5.22696e+06 |      6468732 |         2625.4 |      1669   |            5.8 |
|  1119 |   2014 |       2 | California   | WECC          | cold               | Thursday, February 6, 2014   | 1:05:00 PM          | fuel supply emergency         | Natural Gas             |              160 |                  nan |      6.25992e+06 |      8.70059e+06 |      3.70462e+06 |      1.8728e+07  |     38792291 |         4303.7 |      2124.1 |           12.7 |
|  1324 |   2011 |       6 | Virginia     | RFC           | normal             | Sunday, June 12, 2011        | 7:00:00 PM          | severe weather                | thunderstorm            |              250 |                56000 |      4.14128e+06 |      4.22683e+06 |      1.48658e+06 |      9.8702e+06  |      8110783 |         2265.2 |      1179.2 |           53.3 |
|   415 |   2015 |      12 | Washington   | WECC          | warm               | Wednesday, December 9, 2015  | 4:00:00 AM          | severe weather                | nan                     |              115 |                76300 |      3.89241e+06 |      2.67147e+06 |      2.02896e+06 |      8.59335e+06 |      7170351 |         2380   |      1487.9 |           16.7 |
|   891 |   2013 |       5 | Delaware     | RFC           | normal             | Friday, May 17, 2013         | 8:35:00 AM          | intentional attack            | vandalism               |              nan |                  nan | 282332           | 350320           | 240615           | 873267           |       925353 |         1838.3 |      1083   |           97.3 |


## Exploratory Data Analysis

### Univariate Data Analysis

I started looking at the column for climate category. I wanted to see what the frequency of outages are for each category. And it looks like normal climate category has the most frequency, followed by cold climate.
<iframe 
src="assets/plots/climate_category_frequency.html"
  width="600"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe> 
Then, I wanted to look at the cause category frequencies for each cause.
<iframe
  src="assets/plots/cause_category_frequency.html"
  width="600"
  height="600"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe> 
Seeing that the severe weather have the highest frequency, I wanted to see which cause detail has the highest frequency for only the cause of severe weather. And it turns out to be thunderstorms.
<iframe
  src="assets/plots/cause_category_detail.html"
  width="800"
  height="600"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe> 
Now, I want to look at whether the number of outages change over month. And it looks like most outages occur during summer. This might be due to air conditioning units drawing a significant amount of power, contributing to spikes in electricity demand during hot weather. The sudden increase in demand can exceed the capacity of the electrical infrastructure, leading to outages or brownouts. 
<iframe
  src="assets/plots/outages_per_month.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

### Bivariate Data Analysis

For my bivariate analysis, I wanted to look at customers affected by month, to see if there will be the same pattern as number of outages per month. And there were. There are more customers affected during summer than any other month.

<iframe
  src="assets/plots/month_customer.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>


### Groupby and Aggregate
First, I look at the relationship between cause category and demand. I groupby the cause category, aggregate by the mean of demand loss using

`df.groupby('cause.category')['demand.loss.mw'].mean().reset_index()`

It looks like the overall mean for demand loss is way higher for public appeal. And it looks like demand loss significantly varies between different categories.
<iframe
  src="assets/plots/cause_demand.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

Then, I used the same process for state but by using postal code instead for the sake of plotting my graph:

`demand_loss_by_state = df.groupby('postal.code')['demand.loss.mw'].mean().reset_index()`

It looks like the average demand loss is the highest for Massachusetts.

<iframe
  src="assets/plots/state_demand.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

I also see a clear relationship between month and average demand loss as months from July to December has a much lower demand loss on average. This indicates that it can be useful for predicting demand loss.

`demand_loss_by_cause = ex.groupby('month')['demand.loss.mw'].mean().reset_index()`

<iframe
  src="assets/plots/month_demand.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

## Missingness Analysis

For this portion, I wanted to see the missingness frequency of each column. It turns out that demand loss, the column we are trying to predict, has the highest missingness count, which is followed by cause category.detail and then customers.affected.

|  column |   missingness count |
| ----------- | ----------- |
| demand.loss.mw   | 705 |
| cause.category.detail | 471 |
|customers.affected    | 443 |
| total.sales           |  22 |
| res.sales             |  22 |
| com.sales             |  22 |
| ind.sales             |  22 |
| popden_rural          |  10 |
| popden_uc             |  10 |
| climate.category      |   9 |
| outage.start.date     |   9 |
| outage.start.time     |   9 |
| month                 |   9 |
| popden_urban          |   0 |
| population            |   0 |
| year                  |   0 |
| cause.category        |   0 |
| nerc.region           |   0 |
| postal.code           |   0 |
| u.s._state            |   0 |

### NMAR Analysis

The demand loss column may have missing values if the data was not captured for some outages or if the calculation of demand loss was not feasible in certain cases which would be NMAR. Information about the number of customers affected by an outage may not be available for certain events or may not have been recorded accurately as well leading to all these missing values. For historical data spanning several years, older records may be less complete or have more missing values compared to recent data due to changes in data collection practices or data availability over time. 

I think that cause category detail is probably not available for some cause categories, which would be missing by design, and not NMAR or MAR. But we can analyze it further in the next section.

### Missingness Dependency

For assessing missingness dependency, first, I wanted to see if the missingness of the demand loss column depends on the month.

<iframe
  src="assets/plots/month_missingness_demand.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

It looks like the missingness tend to increase significantly during the summer, which also has higher demand loss on average. 

I also wanted to see if the missingness of demand loss possibly depends on state.

<iframe
  src="assets/plots/state_missingness_demand.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

It looks like some states definitely have more missing values for demand loss than others. We can conduct a hypothesis test to see if this missingness for demand loss actually depends on the state.

#### Testing if the missingess of demand loss depends on State

`Null Hypothesis` : The missingess of the column `demand.loss.mw` does not depend on the state column `u.s._state`

`Alternative Hypothesis` : The missingess of the column `demand.loss.mw` depends on the state column `u.s._state`

I decided to use *TVD* as my test statistic since state is a categorical variable.

`Significane Level` : 0.05

##### *Results*

`Observed TVD` : 8.742915732354533
`P-value` : 0.0

<iframe
  src="assets/plots/state_missingness_demand_test.html"
  width="700"
  height="500"
  frameborder="0"
  margin="0"
  padding="0"
  style="display: block; margin: 0 auto;"
></iframe>

Looking at the results, with a significance level of 0.05, we can reject the null hypothesis that the missing value of demand loss is independent from the state column. This result suggests that the missingness of demand loss might actually be MAR rather than NMAR.

## Hypothesis Testing

In this section, I will be testing if the demand loss changes significantly depending on the cause.category. This will help me determine if the cause category is actually an important variable to use in my baseline/final model of prediction.

`Null Hypothesis` : The column `demand.loss.mw` does not depend on the column `cause.category`

`Alternative Hypothesis` : The column `demand.loss.mw` changes significantly for values in `cause.category`

I decided to use variation between mean demand loss for each state as my test statistic since state is a categorical variable. 

`Significane Level` : 0.05
`Observed Statistic`: 275631.52903857484
`P-value`: 0.046

<iframe
  src="assets/plots/my_plot.html"
  width="700"
  height="500"
  frameborder="0"
  style="display: block; margin: 0 auto;"
></iframe>

With the given results, I reject the null hypothesis that the demand loss column does not depend on cause category. That means cause category will be a useful variable for predicting demand loss in my model.

## Framing A Prediction Problem

In this project, my ultimate goal was to make meaningful analysis and find useful variables for my predictive model. I want to predict the demand loss with columns that I have, which is a regression problem since demand loss column is a continuous numerical variable. I already got rid of the columns that we wouldn't know at the time of prediction such as outage.duration and outage restoration date. This is because we want to be able to predict demand loss before the outage has been resolved. A variable I was concerned to include in my model was `customers.affected`. At the time of prediction, I'm not sure how well the data for customers affected would be collected. But at the same time, it is a very useful variable to include in the model since more customers affected means more demand loss. 

While unexpected events (e.g., natural disasters) may temporarily disrupt data reporting, measures are in place to quickly restore reporting capabilities, minimizing potential disruptions to data availability. Historical data trends indicate that outage incidents and their impacts on customer populations are predictable to a reasonable extent, allowing for reliable forecasting of 'customers.affected' at future time points. This is why I decided to keep this column in my predictive model.

I ended up with the columns I showed above with a number of 19 features available for me to use in the model. Of course, I'm not going to be using all of them to prevent overfitting. But I want to start by experimenting with some of them to see how they perform but also make sense when predicting the demand loss. 

I also want to mention the fact that demand loss column has 700 missing values, meaning that I will have to drop the half of the dataset, which will inevitably affect the performance of my model. Because of this, I used a split of 0.9 to 0.1 training-testing data. I know this has short-comings like overfitting the training data. However, otherwise the model won't have enough data to train with, therefore, won't be able to predict the testing data well enough.

### Baseline Model

I decided to include almost all the columns from above because I think they have meaningful correlations with the demand loss variable. We can see in my previous graphs that demand loss had a clear relationship with cause category, state, and month columns. 

#### Why Are these features valuable?

+ Urban areas with high population density might have different demand patterns and vulnerability to outages compared to rural areas. High population density could lead to higher demand and potentially greater demand loss during an outage. So I decided to include population density for rural, urban and uc.

+ The North American Electric Reliability Corporation (NERC) regions have different grid infrastructures, regulatory environments, and susceptibility to different types of outages. Regional differences can impact the severity and duration of outages, and hence the demand loss.

+ Climate affects both the frequency of outages (e.g., more storms in certain climates) and the demand patterns (e.g., higher demand in hot climates due to air conditioning). Climate can thus be a crucial factor in predicting demand loss.

+ The amount of electricity sold to residential customers can indicate the level of residential demand. Higher residential sales might correlate with higher potential demand loss during outages affecting residential areas. Commercial sales indicate the electricity consumption by businesses. Outages in commercial areas can result in significant demand loss, especially during business hours. Industrial sales also represents electricity consumption by manufacturing and other industrial activities. Outages in industrial areas can lead to substantial demand loss, particularly if the industry is energy-intensive. Total electricity sales give an overall picture of electricity demand. High total sales could mean greater potential for demand loss during outages, as there’s more consumption that can be disrupted.

I tried a lot of things before I came up with this baseline model. I tried using Linear Regression starting with the columns `u.s._state`, `cause.category`, `year`, `month`, and some other columns. But it ended up with terrible R^2 scores for the test set. Then, I decided to use the Random Forest Regressor with additional columns:
`nerc.region`, `climate.category`, `customers.affected`, `total.sales`, `res.sales`, `com.sales`, `ind.sales`, `outage.start`. Additionally, I added an extra column from the `outage.start` column I created called `outage_period_of_day` which indicates whether the outage started in the morning, afternoon, or night. I wanted to add this column because I assumed that probably most outages occur at night, and this might cause the demand loss to be higher when an outage started at night.

| Categorical Value | Columns |
------|-------|
| Nominal | ['u.s._state', 'cause.category', 'month', 'nerc.region', 'climate.category', 'outage_period_of_date'] |
| Ordinal | ['month'] |

| Numerical Value | Columns |
------|-------|
| Discrete | ['year', 'res.sales', 'com.sales', 'ind.sales', 'customers.affected', 'total.sales'] |
| Continuous | ['popden_rural', 'popden_uc', 'outage.start'] |

Here is the graph of predicted Demand Loss vs the actual Demand Loss:

<iframe
  src="assets/plots/baseline.html"
  width="700"
  height="500"
  frameborder="0"
  style="display: block; margin: 0 auto;"
></iframe>

With this model, I got an R^2 score of 0.8795859840297708 for training model and 0.6257468156768007 for testing model. It looks like the data significantly overfits the training model.

### Final Model

Seeing that my baseline model overfit the training set, I decided to use a more robust model, Lasso. However, with this model, I got an R^2 of 0.21747907115757725 for the training set and 0.09967237171570309 for the testing set. Right now, the model significantly underfits the data. 

Instead, I just decied to keep my original model, and add more features by transforming numeric columns with StandardScaler and QuantileTransformer. Got an R^2 of 0.8873545458888363 for training and 0.634079786232939 for testing dataset. This is a slight increase but the data still overfits, and the score is still not good enough. I knew this would be a problem from the beginning, since the Random Forest Regressor tend to overfit the randomness of the data.

Moving on, I decided to use Grid Search for choosing the best hyperparameters and K-Cross Fold Validation to choose the right hyperparameters for my model. 

| Hyperparamter | Values |
|---------------|--------|
| random_forest__n_estimators |  [100, 200, 300] |
| random_forest__max_depth | [None, 10, 20] |
| random_forest__max_features | ['auto', 'sqrt'] |

`Best parameters found by grid search: {'random_forest__max_depth': 10, 'random_forest__max_features': 'auto', 'random_forest__n_estimators': 200}`

`Best CV score: 0.03467264055112714`

`Best Model Testing R^2 score: 0.6293495611491493`

`Best Model Training R^2 score: 0.8494450788515325`

My model has not improved a lot, but I think this is mostly due to the fact that the dataset without the missing values for demand loss is pretty small, which makes the validation sets a little too small to evaluate.

### Fairness Model

I realized that most of the outages are due to severe weather than other categories. This means that we have less data for other causes and our model might not work as good with them. I wanted to see the difference between the RMSEs for each group: where `cause.category` is equal to 'severe weather' or not. I used permutation testing with a 1000 permutations.

RMSE for severe weather: 1050.5669936991242
RMSE for not severe weather: 1118.1947998381281
Observed Difference: 67.62780613900395 

Let's see if this difference is significant. (if our model performs better for severe weather related outages)

`Null Hypothesis` : The difference between the RMSE scores of two groups is not significant, and random

`Alternative Hypothesis` : The difference between the RMSE scores of two groups is significant, and not random

`Significane Level` : 0.05

P-value: 0.461

With the given p-value, we fail to reject the null. Our model is fair for both groups.

<iframe
  src="assets/plots/fairness.html"
  width="700"
  height="500"
  frameborder="0"
  style="display: block; margin: 0 auto;"
></iframe>







