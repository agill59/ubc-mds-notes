# Lecture 6: Evaluation measures and model selection

### Last class

We discussed different ways to evaluate the goodness of fit of a LR and we introduced statistical tests to compare nested models (including the full versus the null model).


### Today:

1. Model Evaluation 

2. Variable Selection

### Lecture 6 Learning Objectives

- Recognize different metrics to evaluate a LR 
- Choose an appropriate measure to evaluate the LR when inference is the primary goal
- Choose an appropriate measure to evaluate the LR when prediction is the primary goal
- Distinguish between measures computed on the training data versus those computed on the test data
- Recognize the limitations of the coefficient of determination (R-square on training set) to evaluate 
- Recognize the limitations of the coefficient of determination to compare nested models
- Choose an appropriate measure to select variables of a LR when inference is the primary goal
- Choose an appropriate measure to select variables of a LR when prediction is the primary goal
- Understand the algorithms (best, forward, backward) to automate a variable selection process in LR
- Interpret the results of a model evaluation and a selection process
- Identify LASSO as a variable selection tool


```R
library(tidyverse)
library(repr)
options(repr.plot.width=7, repr.plot.height=4)
library(ggplot2)
library(broom)
library(gridExtra)
library(moderndive)
library(latex2exp)
library(leaps)

dat <- read.csv("data/Assessment_2015.csv")
dat <- dat %>% filter(ASSESSCLAS=="Residential")  %>% 
        mutate(assess_val = ASSESSMENT / 1000, age=2020-YEAR_BUILT) 


# Our sample
set.seed(561)
dat_s <- sample_n(dat, 1000, replace = FALSE)
```

    ── [1mAttaching core tidyverse packages[22m ──────────────────────── tidyverse 2.0.0 ──
    [32m✔[39m [34mdplyr    [39m 1.1.4     [32m✔[39m [34mreadr    [39m 2.1.5
    [32m✔[39m [34mforcats  [39m 1.0.0     [32m✔[39m [34mstringr  [39m 1.5.1
    [32m✔[39m [34mggplot2  [39m 3.5.1     [32m✔[39m [34mtibble   [39m 3.2.1
    [32m✔[39m [34mlubridate[39m 1.9.3     [32m✔[39m [34mtidyr    [39m 1.3.1
    [32m✔[39m [34mpurrr    [39m 1.0.2     
    ── [1mConflicts[22m ────────────────────────────────────────── tidyverse_conflicts() ──
    [31m✖[39m [34mdplyr[39m::[32mfilter()[39m masks [34mstats[39m::filter()
    [31m✖[39m [34mdplyr[39m::[32mlag()[39m    masks [34mstats[39m::lag()
    [36mℹ[39m Use the conflicted package ([3m[34m<http://conflicted.r-lib.org/>[39m[23m) to force all conflicts to become errors
    
    Attaching package: ‘gridExtra’
    
    
    The following object is masked from ‘package:dplyr’:
    
        combine
    
    


# 1. Model Evaluation

***Have we fit a "good" model?***
    
This is a very general question which we can (and should) break down into more specific questions.
    
    
> Intuitively: if the model is good, $\hat{y}_i$ should be close to $y_i$.


## 1.1 Inference vs. Prediction

To assess or evaluate the model there is a more essencial question to answer: 
    
**What is your goal?**
    
The evaluation metrics are different depending on the goal of the analysis
    
> The estimation methodologies can also differ although we focus on LS in the course
    

- **Inference**: your primary goal is to understand the relation between a response variable $Y$ and a set of explanatory variables $X_1, \ldots, X_p$ 
    
    - you estimate the LR using a sample to understand how variables are associated (in the population)
    
    - you use methods from Inference to draw conclusions about the population from the results obtained in the sample
    


**Examples of inference problems**: 
    
- A real estate agent wants to identify factors that are related to the assessed values of homes (e.g., size of houses, age, amenities, etc) 
    
    
- Biologists want to verify empirically the central dogma of biology that relates mRNA to protein values 
    


- **Prediction**: your primary goal is to make predictions about the response $Y$, and you are not so concerned about how you get those predictions
    
    
    - you estimate the LR using a sample to make predictions of the response for new units (houses, subjects, counties, etc) from the population
    


**Examples of prediction problems**: 
    

- A real estate agent is interested in determining if a house is under- or over-valued given its characteristics (prediction problem)


- Biologists want to use mRNA data to predict protein values of different genes
    

## 1.2 Evaluation metrics

Many metrics used to evaluate LR measure how far $Y$ is from $\hat{Y}$:

**Mean Squared Error**: MSE = $\frac{1}{n}\sum_{i=1}^n(y_i - \hat{y}_i)^2$

- *Training MSE*: if we use the original sample $y_1, \ldots, y_n$ and their predicted values (formula above)

- *Test MSE*: if we predict new responses using the estimated LR (similar formula but averaging the squared errors in the test set) 

**Note**: we can argue that these are *residuals* and not *errors*. However, many software and articles call this measure the MSE.


**Residuals Sum of Squares**: RSS = $\sum_{i=1}^n(y_i - \hat{y}_i)^2$

- sum of the squared residuals (small is good)

- residuals are measured in the *training* set




**Residuals Standard Error**: RSE = $\sqrt{\frac{1}{n-p} \text{RSS}}$
- estimates the standard deviation of the error term $\varepsilon$ (the RSS is divided by the appropriate degrees of freedom to give a "good" estimate of $\sigma = \sqrt{Var(\varepsilon)}$)

- needed to estimate the standard errors of $\hat{\beta}_j$ in classical theory! (for inference)

- $p$ is the number of estimated parameters including the intercept

- a measure based on *training* data to evaluate the fit of the model (for inference)

- gives an idea of the size of the *irreducible* error, very similar to the RSS, small is good

**Note**: again, *errors* is used to defined this measure that is based on *residuals*. 

All the measures listed above are *absolute* measures and it is not easy to judge if these are small enough.

Two important relative measures:

**(i) Coefficient of Determination**, $R^2 = 1 - \frac{RSS}{TSS}$

- it is a common measure of goodness of fit: how well does the model fit the *training* data? 

- it doesn't have a known distribution so you can't use it to test a statistical hypothesis


- it also compares $Y$ vs $\hat{Y}$ using the RSS but relative to the TSS (sum of the squared residuals of the null model, no variables, just an intercept)

- the $R^2 = cor(Y, \hat{Y})^2$, if $\hat{Y}$ is a predition obtained from a LR with an intercept estimated by LS

- the $R^2$ increases as we include more variables in the model so it is not useful to compare models of different sizes

#### Can we compute the $R^2$ on the test set?
    
Yes, as we mentioned for the MSE, the $R^2$ can be computed for new responses in a test set $y_{new}$ compared to the predicted values obtained using the trained LR, $\hat{y}_{new}$.  
    
- Some functions compute the $R^2$ from a validation set or using cross validation (perhaps seen in other courses).   
    
- However, note that it is *no longer the coefficient of determination*. It measures the correlation between the true and the predicted responses in a test set.
 
**Note**: in other courses and articles you may find other measures under the name of $R^2$.  

**(ii) The F-statistic**: $F=\frac{(TSS - RSS)/q}{RSS/(n-p)}$

- tests if there is at least one of the predictors $X_1, X_2, \ldots, X_q$ useful in predicting the response, $H_0: \beta_1 = \beta_2 = \ldots = \beta_q=0$

- compares a full model ($p$ parameters, including the intercept) versus a null (intercept-only) model 

- it also compares $Y$ vs $\hat{Y}$ from the *training* set using the RSS 

- the form of the statistic is such that (asymptotically) it follows an $F$ distribution 

**Class Discussion: Think, Pair, Share**

Take a moment to compare and contrast the $F$-statistic and $R^2$. What is similar about them? What are the differences between them?


- The $F$ statistic and the $R^2$ both depend on the RSS and the TSS so there is a formula that relates them. 

- However, the former has a known $F$ distribution (under certain assumptions) so we can use it to make probabilistic statements.

### Conclusion for Part 1
    
It is important that you understand: 

- what you are computing
    
- which set you are using (training vs test)
    
- what is the purpose of the model    

# 2. Variable Selection

### Do we need all the available predictors in the model?

Some datasets contain *many* variables but not all are relevant and you may want to identify the *most relevant* variables to build a model. But again: 
    
#### What is your goal? (*Inference vs. Prediction*)
    
To decide if a variable (or set of variables) is relevant or not we need to choose an evaluation metric
    
As we have just discussed, the evaluation metric used depends on the goal of the analysis!

## 2.1 Inference

Do all the predictors help to explain the response, or is only a subset of the predictors useful?

Here are some ways we can attempt to answer this question:

- ###  The $F$-test: to compare and test nested models

> `anova`

- ### The $t$- test: to test the contribution of individual variables to explain the response. Thus, we can use these tests to evaluate variables one at a time

> `lm` and `tidy`

*Caution*: if there are many variables in the model (i.e., $p$ is large) using individual $t$-tests may result in many false discoveries (i.e., reject a true $H_0$, by chance)



- ### The $R^2$ (or the RSS):

The RSS decreases as more variables are included in the model! 

> You can use the $R^2$ or the $RSS$ to compare models of *equal sizes*



- ### The adjusted $R^2$:
    
To overcome this problem the **$R^2$** is penalized by the number of variables in the model ($p$):

$$ \text{adjusted } R^2 = 1- \frac{RSS/(n-p)}{TSS/(n-1)} $$ 

- Note that the RSS is penalized by the degrees of freedom. 

- You can use the adjusted $R^2$ to compare models of *different sizes* (not necessarily nested)  

Caution note: the *training* set is used (over an over) to select so it can't be used again to assess the final significance of the model. 
    
> This problem is known as the *post-inference* problem

**Question**

The coefficient of determination, $R^2= 1- RSS/TSS$ can be used to compare the fit of two nested models.  True or False?

A. TRUE

B. FALSE

### Example: 

- Suppose we compare a model with 2 explanatory variables with another model with 3 additional variables, using $n=100$ observations. 

- The RSS of the larger model (RSS5) are smaller than those of the reduced models (RSS2). 

- The adjusted $R^2$ divides the smaller RSS5 by 94 (100-5-1) and larger RSS2 by 97 (100-2-1), so it is not obvious anymore which of the adjusted ones will be smaller.

- It can go either way depending on how relevance of the additional variables to remarkably decrease the residuals.


```R
fit2 <-  lm(assess_val ~ age + BLDG_METRE , data=dat_s)

fit5 <-  lm(assess_val ~ age + BLDG_METRE + FIREPLACE + GARAGE + BASEMENT , data=dat_s)


glance(fit2)
glance(fit5)
```


<table class="dataframe">
<caption>A tibble: 1 × 12</caption>
<thead>
	<tr><th scope=col>r.squared</th><th scope=col>adj.r.squared</th><th scope=col>sigma</th><th scope=col>statistic</th><th scope=col>p.value</th><th scope=col>df</th><th scope=col>logLik</th><th scope=col>AIC</th><th scope=col>BIC</th><th scope=col>deviance</th><th scope=col>df.residual</th><th scope=col>nobs</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>0.6690865</td><td>0.6684227</td><td>108.674</td><td>1007.936</td><td>3.781472e-240</td><td>2</td><td>-6105.789</td><td>12219.58</td><td>12239.21</td><td>11774610</td><td>997</td><td>1000</td></tr>
</tbody>
</table>




<table class="dataframe">
<caption>A tibble: 1 × 12</caption>
<thead>
	<tr><th scope=col>r.squared</th><th scope=col>adj.r.squared</th><th scope=col>sigma</th><th scope=col>statistic</th><th scope=col>p.value</th><th scope=col>df</th><th scope=col>logLik</th><th scope=col>AIC</th><th scope=col>BIC</th><th scope=col>deviance</th><th scope=col>df.residual</th><th scope=col>nobs</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>0.678893</td><td>0.6772777</td><td>107.2131</td><td>420.3082</td><td>2.991024e-242</td><td>5</td><td>-6090.748</td><td>12195.5</td><td>12229.85</td><td>11425676</td><td>994</td><td>1000</td></tr>
</tbody>
</table>



## 2.2 Prediction

Do all the predictors help to *predict* the response of new *test* samples? or is only a subset of the predictors useful?

As we discussed before, we need a different metric to evaluate, the $F$-test can not be used to compare *out-of-sample* predictions from different models!

- ### The test MSE

> if an independent test set is not available, we can use cross-validation to generate *test* sets

#### Caution note: the set that has been used to *select* can not be used to evaluate the prediction of the selected model

In the past, a cross-validation study was not always feasible, in particular for large datasets. Thus, different measures were proposed to "approximate" the test MSE.



- ### Estimates of the test MSE

The **Mallow's Cp, Akaike information criterion (AIC) and Bayesian information criterion(BIC)** add different penalties to the training RSS to adjust for the fact that the training error tends to underestimate the test error

> the penalty increases as the number of predictors in the model increases 

> these quantities can be computed for models more complex than LR (more in Regression II)

### An automated proceedure

When we don't have any idea about which variables should be included in the model. Ideally, you want to select the best model out of *all possible models* of all possible sizes. 

> For example: if the dataset has 2 explanatory variables $X_1$ and $X_2$, there are 4 models to compare: (1) an intercept-only model, (2) a model with only $X_1$, (3) a model with only $X_2$, and (4) a model with both $X_1$ and $X_2$. 



However, the number of *all possible* models become too large rapidely, even for small subset of variables

> there are a total of $2^p$ models from a set of $p$ variables

> if $p = 20$ (20 available explanatory variables) we need to evaluate more than a million models! 

There are methods to search more efficiently for a good model (although it may not find the "best" one out of all possible):

- **Forward selection**: 
Image from [ISLR](http://faculty.marshall.usc.edu/gareth-james/ISL/ISLR%20Seventh%20Printing.pdf)

<img src="img/forward.png" style="width:500px;"/>

1. Start with the intercept only model: $y_i = \beta_0 + \varepsilon_i$

- Remember that $\hat{\beta}_0 = \bar{y}$ from the training samples, so $\hat{y}_{0} = \bar{y}$ for any observation (from the training or the test set)

2. Select the best model of each size (without counting the intercept)

**Size 1** Evaluate all models of size 1, choose the best model of size 1 (based on RSS, equal size models), call it $\mathcal{M}_1$. 

$\vdots$


**Size 2** Starting with the best size 1 model, add 1 variable and evaluate all the resulting models of size 2. Choose the best model of size 2 (based on RSS), call it $\mathcal{M}_2$ (note *all* these models share 1 variable selected in previous step)


$\ldots$ continue until you reach the full model


**Size p** there's only one full model, call it $\mathcal{M}_p$.

Note that we can stop this iteration earlier if we want a model of a predetermined size


3. Now we have to select the best out of $p$ models: $\mathcal{M}_1$ (the best model of size 1), $\mathcal{M}_2$ (the best model of size 2 given the selection in previous step), $\ldots, \mathcal{M}_p$ (the full model of size $p$)

> You can't use the RSS to compare models of different sizes, but you can use a cross-validation or validation MSE, the $C_p$ (proportional to AIC), the BIC, or the adjusted $R^2$ depending on the data available and the goal of the study.

Other selection procedures include:

- **Backward selection**: start with the full model and remove variables, one at a time


- **Hybrid selection**: after adding a variable, the method may also remove variables 


### Example

- Forward selection with the assessment value data via the `regsubsets()` function in the package `leaps`

- Selection Criteria: Mallows $C_p$

- Note: The best model is the one that shows the smallest $C_p$


```R
# Split data into train/test sets 

set.seed(561)

dat_train <- sample_n(dat_s, size=nrow(dat_s)*0.75, replace=FALSE)
dat_test <- anti_join(dat_s, dat_train, by="the_geom")
```


```R
# note: you could use lm(y~.) notation if you want to consider all possible explanatory variables 

dat_forward <- regsubsets(
assess_val ~ age + BLDG_METRE + FIREPLACE + GARAGE + BASEMENT, 
data = dat_train, nvmax = 5, method = "forward",
)

fwd_summary <- summary(dat_forward)

fwd_summary

fwd_summary$cp
```


    Subset selection object
    Call: regsubsets.formula(assess_val ~ age + BLDG_METRE + FIREPLACE + 
        GARAGE + BASEMENT, data = dat_train, nvmax = 5, method = "forward", 
        )
    5 Variables  (and intercept)
               Forced in Forced out
    age            FALSE      FALSE
    BLDG_METRE     FALSE      FALSE
    FIREPLACEY     FALSE      FALSE
    GARAGEY        FALSE      FALSE
    BASEMENTY      FALSE      FALSE
    1 subsets of each size up to 5
    Selection Algorithm: forward
             age BLDG_METRE FIREPLACEY GARAGEY BASEMENTY
    1  ( 1 ) " " "*"        " "        " "     " "      
    2  ( 1 ) "*" "*"        " "        " "     " "      
    3  ( 1 ) "*" "*"        " "        " "     "*"      
    4  ( 1 ) "*" "*"        " "        "*"     "*"      
    5  ( 1 ) "*" "*"        "*"        "*"     "*"      



<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>23.3035072471821</li><li>12.8565291757996</li><li>5.86568771125292</li><li>4.70146960044644</li><li>6</li></ol>




```R
fwd_summary_table <- data.frame(
    Model = 1:5,
    RSQ = fwd_summary$rsq,
    ADJ.RSQ = fwd_summary$adjr2,
    RSS = fwd_summary$rss,
    Cp = fwd_summary$cp,
    BIC = fwd_summary$bic
    )

fwd_summary_table
```


<table class="dataframe">
<caption>A data.frame: 5 × 6</caption>
<thead>
	<tr><th scope=col>Model</th><th scope=col>RSQ</th><th scope=col>ADJ.RSQ</th><th scope=col>RSS</th><th scope=col>Cp</th><th scope=col>BIC</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1</td><td>0.6906317</td><td>0.6902181</td><td>7144086</td><td>23.303507</td><td>-866.6769</td></tr>
	<tr><td>2</td><td>0.6956371</td><td>0.6948222</td><td>7028498</td><td>12.856529</td><td>-872.2907</td></tr>
	<tr><td>3</td><td>0.6992527</td><td>0.6980433</td><td>6945005</td><td> 5.865688</td><td>-874.6334</td></tr>
	<tr><td>4</td><td>0.7005252</td><td>0.6989173</td><td>6915621</td><td> 4.701470</td><td>-871.1933</td></tr>
	<tr><td>5</td><td>0.7008073</td><td>0.6987966</td><td>6909107</td><td> 6.000000</td><td>-865.2800</td></tr>
</tbody>
</table>




```R
coef(dat_forward, 4)
coef(dat_forward, 5)
```


<style>
.dl-inline {width: auto; margin:0; padding: 0}
.dl-inline>dt, .dl-inline>dd {float: none; width: auto; display: inline-block}
.dl-inline>dt::after {content: ":\0020"; padding-right: .5ex}
.dl-inline>dt:not(:first-of-type) {padding-left: .5ex}
</style><dl class=dl-inline><dt>(Intercept)</dt><dd>50.9525743083722</dd><dt>age</dt><dd>-0.680575404742103</dd><dt>BLDG_METRE</dt><dd>2.44501212626386</dd><dt>GARAGEY</dt><dd>25.7743017375825</dd><dt>BASEMENTY</dt><dd>64.6837216663365</dd></dl>




<style>
.dl-inline {width: auto; margin:0; padding: 0}
.dl-inline>dt, .dl-inline>dd {float: none; width: auto; display: inline-block}
.dl-inline>dt::after {content: ":\0020"; padding-right: .5ex}
.dl-inline>dt:not(:first-of-type) {padding-left: .5ex}
</style><dl class=dl-inline><dt>(Intercept)</dt><dd>45.5291054528736</dd><dt>age</dt><dd>-0.638775834961519</dd><dt>BLDG_METRE</dt><dd>2.4332424185864</dd><dt>FIREPLACEY</dt><dd>8.28400506276642</dd><dt>GARAGEY</dt><dd>24.3276191625825</dd><dt>BASEMENTY</dt><dd>65.1444860050404</dd></dl>



However, a word of caution regarding stepwise model selection: although widely discussed, these approaches have many potential issues such as multiple comparison problems resulting in $p$-values that are too low. See page 68 of [Regression Modelling Strategies](https://warin.ca/ressources/books/2015_Book_RegressionModelingStrategies.pdf) for a more detailed list of problems associated with stepwise model seleciton. 

# 3. LASSO

Regularization offers an alternative way to select a model by penalizing the RSS.

- For example: ridge, lasso, elastic net

- These are different *shrinkage estimation methods* that shrink coefficient estimates towards zero 

- Alternatives to variable selection with $p>n$ and it is less time consuming

LASSO (also called L1-regularization) is a very popular approach for variable selection. The model minimizes

$$ \sum_{i=1}^n (y_i-\sum_{j=1}^p  x_{ij}\beta_j))^2+\lambda \sum_{j=1}^p |\beta_j|,$$

where $\lambda$ is the penalty term. Note that when $\lambda=0$ we are left with a least squares regression model. In R, the `glmnet` package is commonly used to perform LASSO regression. 

### In-Class Exercise

Using the teaching score data from UT Austin from last class, perform backwards model selection to select the "best" model based on Mallow's $C_p$. Think about whether we can/should assess the interaction model using `regsubsets()`.


```R
# Split data into train/test sets 
set.seed(561)

evals_train <- sample_n(evals, size=nrow(evals)*0.75, replace=FALSE)
evals_test <- anti_join(evals, evals_train, by="ID")
```


```R
dat <- evals_train %>%
  mutate(sex = gender) %>%
  select(score, age, sex)

head(dat)
```


<table class="dataframe">
<caption>A tibble: 6 × 3</caption>
<thead>
	<tr><th scope=col>score</th><th scope=col>age</th><th scope=col>sex</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><td>4.7</td><td>38</td><td>female</td></tr>
	<tr><td>5.0</td><td>50</td><td>male  </td></tr>
	<tr><td>3.5</td><td>52</td><td>female</td></tr>
	<tr><td>4.4</td><td>47</td><td>female</td></tr>
	<tr><td>4.0</td><td>51</td><td>female</td></tr>
	<tr><td>4.1</td><td>52</td><td>female</td></tr>
</tbody>
</table>




```R
# YOUR ANSWER HERE
```


    Subset selection object
    Call: regsubsets.formula(score ~ age + sex, data = dat, nvmax = 3, 
        method = "backward", )
    2 Variables  (and intercept)
            Forced in Forced out
    age         FALSE      FALSE
    sexmale     FALSE      FALSE
    1 subsets of each size up to 2
    Selection Algorithm: backward
             age sexmale
    1  ( 1 ) "*" " "    
    2  ( 1 ) "*" "*"    



<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>8.59622096211768</li><li>3</li></ol>



#### Summary: what have we learned today?
