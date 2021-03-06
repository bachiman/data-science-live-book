

Feature Engenieering
===

### What is this about?

**What are we going to review in this chapter?**


### Removing outliers 

**Model building**: Some models such as random forest and gradient boosting machines tend to deal better with outliers, but some noise will affect results anyway. 

**Communicating results:** If we need to report the variables used in the model, we'll end up removing outliers to not see an histogram with only one bar, and/or show not a biased mean. 

It's better to show a non-biased number than justifying the model _will handle_ extreme values.

**Type of outliers:** 

* Numerical: For example the ones which bias the mean.
* Categorical: Having a variable in which the dispersion of categories is quite high (high cardinallity). For example: postal code.

Jump to <a href="http://livebook.datascienceheroes.com/data_preparation/outliers_treatment.html">Outliers Treatment</a> chapter.

<br>

### Don't use information from the future

<img src="back_to_the_future.png" width='250px'> 

Common mistake when starting a new predictive model project, for example:

Imagine we need to build a predictive model to know what users are likely to adquire full subscription in a web application, and this software has a ficticious feature called it `feature_A`:



| user_id|feature_A |full_subscription |
|-------:|:---------|:-----------------|
|       1|yes       |yes               |
|       2|yes       |yes               |
|       3|yes       |yes               |
|       4|no        |no                |
|       5|yes       |yes               |
|       6|no        |no                |
|       7|no        |no                |
|       8|no        |no                |
|       9|no        |no                |
|      10|no        |no                |


We build the predictive model, we got a perfect accuracy, and an inspection throws the following: _"100% of users that have full subscription, uses Feature A"_. Some predictive algorithms report variable importance, thus `feature_A` will be at the top.

**The problem is:** `feature_A` is only availble **after the user goes for full subcription**. Therefore it cannot be used.

**The key message is**: Don't trust in perfect variables, nor perfect models. 

<br>

### Play fair with data, let it to develop their behavior

Like in nature, things have a minimun and maximun time to start showing certain behavior. This time oscilates from 0 to infinite. In practice it's recommended to study what is the best period to analyze, in other words, we may exclude all the behavior before and after this observation period. To stablish cuts to data it's not so straight forward since it may be kind of subjective.

Imagine we've got a **numerical variable** which increases as time moves. We may need to define an **observation time window** to filter the data and feed the predictive model. 

* Setting the **minimun** time: How much time is it need to start seeing the behavior? 
* Setting the **maximun** time: How much time is it needed to see the end of the behavior? 

Eassiest solution is: setting minimun since begin, and the maximun as the whole history.


<font color="#5d6d7e">

**Case study:** 

Two people, `Ouro` and `Borus`, are users of a web application which has certain functionalty called `feature_A`, and we need to build a predictive model which forecast based on `feature_A` usage  -measured in 'clicks", if the person is going to adcquire `full_subscription`.

The current data says: Borus has `full_subscription`, while Ouro doesn't. 

<img src="figure/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="400px" />


User `Borus` starts using `feature_A` from day 3, and after 5 days she has more use -15 vs 12 clicks- on this feature than `Ouro` who started using it from day 0. 

If `Borus` adcquiere `full subscription` and `Ouro` doesn't, _what would the model will learn?_ 

When modeling with full history -`days_since_signup = all`-, the higher the `days_since_signup` the higher the likelihood, since `Borus` has the highest number.

However, if we keep only with the user history corresponding to their first 5 days since signup, the conclussion is the opposite. 

_Why to keep first 5 days of history?_

The behavior in this kick-off period may be more relevant -in terms of prediction accuracy- than analyzing the whole history. It depends on each case as we said before. 

</font>

<br> 

### Creating new variables 

Contiue last section example, a really important matter is to create new variables that reflect different stories from data. 

Other ideas to handle time variables:

* 
* Create two variables: quantity of clicks on first 5 days since signup, and quantity of clicks on the period from 6 to 30 days.
* Last point is considering the begining, but we can capture the history up to now: quantity of clicks during last month, during last 3 months, etc.
* Average variables per time: `monthly_clicks_average`.

<br>

### Converting categorical variables into numerical 

Categorical, nominal or string variable are necessary for some algorithms to work, they don't handle other data type. Other situation could be when plotting certain graph and we want all the variables at the same time, for example in correlation plot.

In R using `caret` package this task is straitforward. This converting every categorical variable into a flag one, also known as _dummy_ variable. After the conversion original categorical

If the original categorical variable has 2 possible values, it will result in 30 new columns holding the value `0` or `1`, when `1` represents the presence of that category in the row.

If we use package `caret` from R, this conversion only takes two lines of code:


```r
library(caret) # contains dummyVars function
library(dplyr) # data munging library
library(funModeling) # df_status function
  
## Checking categorical variables
status=df_status(heart_disease, print_results = F)
filter(status,  type %in% c("factor", "character")) %>% select(variable)
```

```
##              variable
## 1              gender
## 2          chest_pain
## 3 fasting_blood_sugar
## 4     resting_electro
## 5                thal
## 6        exter_angina
## 7   has_heart_disease
```

```r
## It converts all categorical variables (factor and character) into numerical
## skipping the original so the data is ready to use
dmy = dummyVars(" ~ .", data = heart_disease)
heart_disease_2 = data.frame(predict(dmy, newdata = heart_disease))

# Checking the new numerical data set:
colnames(heart_disease_2)
```

```
##  [1] "age"                    "gender.female"         
##  [3] "gender.male"            "chest_pain.1"          
##  [5] "chest_pain.2"           "chest_pain.3"          
##  [7] "chest_pain.4"           "resting_blood_pressure"
##  [9] "serum_cholestoral"      "fasting_blood_sugar.0" 
## [11] "fasting_blood_sugar.1"  "resting_electro.0"     
## [13] "resting_electro.1"      "resting_electro.2"     
## [15] "max_heart_rate"         "exer_angina"           
## [17] "oldpeak"                "slope"                 
## [19] "num_vessels_flour"      "thal.3"                
## [21] "thal.6"                 "thal.7"                
## [23] "heart_disease_severity" "exter_angina.0"        
## [25] "exter_angina.1"         "has_heart_disease.no"  
## [27] "has_heart_disease.yes"
```

Original data `heart_disease` has been converted into `heart_disease_2` with no categorical variables. There are only numerical and dummy. Note every new variable has a _dot_ following by the _value_.

If we check the before and after for the 7th patient (row) in variable `chest_pain` which can take the values `1`, `2`, `3` or `4`:


```r
# before
as.numeric(heart_disease[7, "chest_pain"])
```

```
## [1] 4
```

```r
# after
heart_disease_2[7, c("chest_pain.1", "chest_pain.2", "chest_pain.3", "chest_pain.4")]
```

```
##   chest_pain.1 chest_pain.2 chest_pain.3 chest_pain.4
## 7            0            0            0            1
```

Having keept and transformed only numeric variables, excluding the nominal ones, the data `heart_disease_2` is ready to be used.

<br>


### Is it categorical or numerical? Think about it

Consider `chest_pain` variable, which can take values `1`, `2`, `3` or `4`. Is this variable categorical or numerical?

If the values are ordered, it can be considered as numerical since it exhibits an **order**, i.e. 1 is less than 2, 2 is less than 3 and 3 is less than 4. 

If we create a decision tree model, we would expect to have rules like: "`If chest_pain > 2.5 then...`". Does it makes sense? The algorithm is splitting the variable by a value that is not present, 2.5, but the interpretation is "If the value is between 2 and 3".  

From the analysis point of view it can be tricky. The rule has the "`>`" symbol, so it is capturing all the `chest_pain >= 3`.

<br>

#### Is the solution to treat all as categorical?

No... A numerical variale carries more information than a nominal one because of its order. In categorical variables values cannot be compared. Let's say it's not possible to do a rule like `If postal code is > "AX2004-P"`.

Values of nominal variable can be compared, if we have a another variale to use as a reference (usually the an outcome ot predict). 

Also there is the **high cardinallity** issue in categorical variable, for example a `postal code` variable containinig hundreds of different values. This book addressed it in both chapters: handling high categorical variable for <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_descriptive_stats.html" target="blank">descriptive statistics</a> and when we do <a href="http://livebook.datascienceheroes.com/data_preparation/high_cardinality_predictive_modeling.html" target="blank">predictive modelling</a>.

Anyway, you can do the _free test_ of converting all variables into categorical and see what happen. Comparing the results vs. keeping the numerical. Remember to use some good error measure for the test like Kappa or ROC statistic, and to cross-validate the results.

<br>

#### Be aware of directly convert categorical into numerical

Imagine we have a categorical variable, and we need to convert into one numerical. Same case as before, but trying a different **transformation**: assiging a different number to each category.

We have to be careful when doing such transformation because we are **introducing order** to the variable. 

Considering the following data example, having 4-rows. The first two-variables are `visits` and `postal_code` this play either as two input variables or `visits` as input and `postal_code` as output.

Following code will show the `visits` depending on `postal_code` transformed according to two criteria:

* `transformation_1`: Assign a sequence number based on the given order.
* `transformation_2`: Assign a number based on the amount of `visits`.


```r
# creating data -toy- sample 
df_pc=data.frame(visits=c(10, 59, 27, 33), postal_code=c("AA1", "BA5", "CG3", "HJ1"), transformation_1=c(1,2,3,4), transformation_2=c(1, 4, 2, 3 ))

# printing table
knitr::kable(df_pc)
```



| visits|postal_code | transformation_1| transformation_2|
|------:|:-----------|----------------:|----------------:|
|     10|AA1         |                1|                1|
|     59|BA5         |                2|                4|
|     27|CG3         |                3|                2|
|     33|HJ1         |                4|                3|

```r
library(gridExtra)

# transformation 1
plot_1=ggplot(df_pc, aes(x=transformation_1, y=visits, label=postal_code)) +  geom_point(aes(color=postal_code), size=4)+ geom_smooth(method=loess, group=1, se=FALSE, color="lightblue", linetype="dashed") + theme_minimal()  + theme(legend.position="none") + geom_label(aes(fill = factor(postal_code)), colour = "white", fontface = "bold")
  

# transformation 2
plot_2=ggplot(df_pc, aes(x=transformation_2, y=visits, label=postal_code)) +  geom_point(aes(color=postal_code), size=4)+ geom_smooth(method=lm, group=1, se=FALSE, color="lightblue", linetype="dashed") + theme_minimal()  + theme(legend.position="none") + geom_label(aes(fill = factor(postal_code)), colour = "white", fontface = "bold")
  
# arranging plots side by side
grid.arrange(plot_1, plot_2, ncol=2)
```

<img src="figure/unnamed-chunk-6-1.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" width="400px" />

For sure, no body does a predictive model with only 4 rows, but the intention of this example is to show how the relationship changes from non-linear (`transformation_1`) to another linear (`transformation_2`).  Making the things easier to the predictive model, as well as explaining the relationship.

This effect is the same when we handle millions of rows and the number of variables scale to hundreds. Learning from small data is a good approach in these cases.

<br>

<br>

### Custom buckets

It's common to create custom binning or custom buckets in the **Feature Engineering** stage. These buckets can be created based on the ingformation provided by `cross_plot`.


```r
# Lock5Data contains many data sets to practice. One of them: HollywoodMovies2011
library(Lock5Data)
data("HollywoodMovies2011")

HollywoodMovies2011$TheatersOpenWeek_2=cut(HollywoodMovies2011$TheatersOpenWeek, breaks = c(0, 2995, 3400, 4375), dig.lab = 9)

freq(HollywoodMovies2011, "TheatersOpenWeek_2")
summary(HollywoodMovies2011$TheatersOpenWeek_2)
```
