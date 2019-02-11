# life Expectancy in the US

## Goal

To inspect what variables have a statistically significant affect on life expectancy in the United States.

## ETL

 To gather data we looked at several different data sources on a county level from 2010 in the United States. 2010 was a census year for the United States, which allowed us to find several datasets from government and peer-reviewed sources for feature selection including
* Minimum wages by state from the [National Conference of State Legislatures](https://www.dol.gov/whd/minwage/mw-consolidated.htm)
* Obesity data from the [Center for Disease Control](https://www.cdc.gov/diabetes/data/countydata/countydataindicators.html)
* Smoking Data from BioMed Central’s open-access peer-reviewed online journal [Population Health Metrics](https://pophealthmetrics.biomedcentral.com/articles/10.1186/1478-7954-12-5)
* Food stamps data from the [United States Department of Agriculture](https://www.fns.usda.gov/pd/supplemental-nutrition-assistance-program-snap)
* Unemployment data from the [United States Department of Agriculture](https://www.ers.usda.gov/data-products/county-level-data-sets/download-data/)

Finally, we got the average Life Expectancy by county from [Global Health Data Exchange](http://ghdx.healthdata.org/us-data ) for our target variable.

Since the data came from 5 different sources we added a state and county ID column to each data set to consistently join the data (e.g. there were 31 “Washington” counties). The individual cleaned data frames, as well as the final data frame, can be found in the data_files folder as .csv files.

## EDA & FEATURE ENGINEERING

**Confirming Distributions**

In order to run a linear regression analysis, our data needed to meet three criteria:
* They needed to be normally distributed
* They needed to be linear
* They needed homoskedastic

![screen shot 2019-02-11 at 9 54 44 am](https://user-images.githubusercontent.com/39356742/52571730-29e24980-2de4-11e9-96bf-f85b6dc0a86c.png)

We found that the majority of our data was normally distributed, with the exception of food stamps (SNAP) data and minimum-wage data.
* The distribution of the minimum wage is due to the fact that it was on the state-level while the rest of our data was on the county-level. To make this feature more workable, we decided to turn it into a categorical variable indicating whether each state’s minimum wage was above or below the mean minimum wage of the data as a whole.
* To normalize the SNAP data we used a log-transform, which helped it become less skewed and have lower variability.

![screen shot 2019-02-11 at 10 09 31 am](https://user-images.githubusercontent.com/39356742/52572213-4b900080-2de5-11e9-8b4a-191bf9bca6f2.png)

**Linearity & Homoskedasticity**

To confirm linearity and homoscedasticity, we inspected scatter plots of each of our independent variables with our dependent variable.

![screen shot 2019-02-11 at 10 29 17 am](https://user-images.githubusercontent.com/39356742/52573349-f3a6c900-2de7-11e9-9a3b-5d4e6ebc1ae1.png)

Notes on interpreting the scatter plots: 
* Homoskedasticity essentially means that all of the data points within our features have the relatively the same variance. If your data is heteroskedastic, you'll run the risk of overfitting your model. If our features are homoskedastic, they will be scattered across a scatter plot in a consistently-dense way (i.e. the dots would not form a horizontal cone shape in either direction).
* If our features have linear relationships with our dependent variable (Life Expectancy), our scatters will form a line cluster going either in the positive or negative direction.

We found that most of our data was both linear and homoskedastic.

**Categorical variables**

To account for states in the model, and use them as possible features, we created a categorical variable for each state.

**Scaling**

To perform analysis and fit models on the data, we also wanted to scale our continuous variables using a Standard Scaler from the scikit-learn library.

**Correlations**

Finally, we inspected each features’ correlations to one another.

First with the initial features.
![screen shot 2019-02-11 at 10 44 56 am](https://user-images.githubusercontent.com/39356742/52574547-89dbee80-2dea-11e9-9eef-027d932ad61e.png)

As well as the states.
![screen shot 2019-02-11 at 10 45 27 am](https://user-images.githubusercontent.com/39356742/52574589-a5df9000-2dea-11e9-8c2f-fa9a1f93bf3d.png)

## BUILDING BASE MODELS

**Statsmodels**

We chose both to run an OLS regression using the Statsmodels library as well as a linear regression with the Scikit-Learn library, wanting to compare the performance between the two.

![screen shot 2019-02-11 at 10 55 37 am](https://user-images.githubusercontent.com/39356742/52575100-9f9de380-2deb-11e9-9849-0e9da1c09d0f.png)

From our Statsmodels’ OLS we got an r-squared value of approximately 80%, which is the percent variance explained by our model. 

We observed that our data was slightly negatively skewed at -0.285 and leptokurtic at 5.619, the latter meaning that we had some outlier data points, or occasional values exceeding (in terms of standard deviations from the mean) what was predicted by the normal distribution. None of these were extreme enough for us to further tweak our algorithm at this point.

From this model, we also observed that our SNAP data, as well as the categorical variables we created for AZ, CO, CT, FL, GA, ME, MA, MS, RI, TX, UT all have p-values > 0.05, meaning that there is no statistically significant relationship between these variables and our target variable (life expectancy).

We visualized the model’s predictions versus the true y-values from our test dataset to evaluate how accurate our model is, and got a MSE of 0.17352770075852306 and RMSE of 0.4165665622184803.

![screen shot 2019-02-11 at 10 57 58 am](https://user-images.githubusercontent.com/39356742/52575295-ef7caa80-2deb-11e9-87a6-fab8f1a847c6.png)

**Scikit-Learn**

After running a linear regression with sklearn, we got the same r-squared value (0.799) and found no predictive  difference between the libraries.

## FEATURE SELECTION & FURTHER ENGINEERING

**Feature Selection**

After running the initial linear regression model (using Scikit-Learn), we did some feature selection and engineering, starting with the wrapper method to select the top features of our model.
Then we used two filter methods
* Variance Threshold
* Univariate Feature Selection

**Further Feature Engineering**

Using polynomial terms to add features to the model we ended with 1,539 features. This model is raised the R-squared value, but decreased the adjusted R-squared value. 

**LASSO Method**

Using the "Least Absolute Shrinkage and Selection Operator" or LASSO to fit a model. LASSO is similar to Ordinary Least Squares although it performs both L1 regularization and selects features. We tried several alpha values (which is the constant that multiplies the L1 term) to optimize for the best R-squared value. An interesting note, if alpha is equal to 0, then the model is functionally a Ordinary Least Squares model, which was one of our better performing models.

![screen shot 2019-02-11 at 11 12 03 am](https://user-images.githubusercontent.com/39356742/52576346-e1c82480-2ded-11e9-8b96-e083634de9fe.png)

This graphs shows the features LASSO kept and their corresponding weight in the model. 

## FINAL THOUGHTS

The initial linear regression using Sklearn is the best performing model with a R-squared value of .799 and a Test Mean Squared Error of .174. 

![screen shot 2019-02-11 at 11 32 07 am](https://user-images.githubusercontent.com/39356742/52577718-b266e700-2df0-11e9-814c-d1a92e6389c6.png)

We would like to add more features to increase the predictive power of our model (including Medicare, Educational Attainment, and Small Area Income), however we were not able to find data sets for either 2010 or for a county level in 2010. Also, we could further feature engineer by incorporating polynomial features into other regression models.


