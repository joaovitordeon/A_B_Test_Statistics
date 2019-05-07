# Analyze A/B Test Results

Table of Contents:
- Introduction
- Part I - Probability
- Part II - A/B Test
- Part III - Regression

<br>

## Introduction
**A/B tests** are very commonly performed by data analysts and data scientists. We will be working to understand the results of an **A/B test** run by an **e-commerce website**.
Our goal is to help the company understand if they should implement the new page, keep the old page, or perhaps run the experiment longer to make their decision. Let's get this!

The whole process can be found in file :`Analyze_ab_test_results_notebook.ipynb`.

## Part I - Probability

Here is a sample of our table:


|    user_id      |         timestamp	        |     group       |  landing_page  | converted |
|-----------------|-----------------------------|-----------------|----------------|-----------|
|     864975      | 2017-01-21 01:52:26.210827  |     control     |    old_page    |     0     |
|     853541      | 2017-01-08 18:28:03.143765	|     treatment   |    new_page	   |     1     |

- *user_id* : user id of A/B test
- *timestamp* : time the test happened
- *group* : group can be control or treatment
- *landing_page* : page that user is testing( old page or new page)
- *converted* : 1 if the user has been converted to new changes or 0 if the user has not.

*After some cleaning in our data:*
1. The number of unique users in the dataset: **290584**
2. Given that an individual was in the **control** group, the probability they converted is: **0.1204**
3. Given that an individual was in the **treatment** group, the probability they converted is: **0.1188**
4. The probability of an individual converting regardless of the page they receive is: **0.119597**
5. The probability that an individual received the **new page**: **0.50006**

***There is no evidence that a individual tends to get the new page as the yours favorite
 The probability of treatment group was even smaller than control group, considering the proportion of both sizes groups are almost equals ,50% for each, approximately.***

## Part II - A/B Test

 If we want to assume that the old page is better unless the new page proves to be definitely better at a Type I error rate of 5%(*alpha*), these are our null and alternative hypotheses:<br>
**Ho:Pnew <= Pold<br>
H1:Pnew > Pold**<br>
where,**Pnew** and **Pold** are converted rates for the old and new pages.
<br>
 Now we're going to perform the sampling distribution for the difference in converted rate between the two pages over 10,000 iterations(using the *bootstrap* method) of calculating<br>
 an estimate from the null(**Pnew-Pold<=0**).<br>

 With 10000 iterations, we have a *normal distribution* like this:<br>
![ALT](/pics/normal1.png "Ho distribution")
<br>
***This plot shows that old values are greater than new values in most of time***

----
Projecting the mean of the vector of differences over a normal distribution with center equal to zero (Ho: Pnew - Pold <= 0) and standard deviation equal to that of the<br> difference vector,<br>
we obtain this graph:<br>
![ALT](/pics/normal2.png "normal with marked mean of vector of differences")
<br>
***Its `P-value = 0.9004` and thats suggests that we should not reject the null hypothesis***

----
Now we're going to simulate 10000 **Pnew-Pold** values using this same process similarly to the one we calculated above, but using the `numpy.random.choice` under the *converted* feature. <br>
Again, our plot is a normal distribution:<br>
![ALT](/pics/normal3.png "Ho distribuition with choice function")
<br>
***If we analyze, this graph looks quite like the first***

----
Doing the normal distribution again the same in figure two, but with the new vector made with the `numpy.random.choice` we obtain this:<br>
![ALT](/pics/normal4.png "normal with marked mean of vector of differences with choice function")
<br>
***We just computed the p-value, which is the probability of getting the statistic test, or a more extreme value, under the null hypothesis.<br>
With *alpha* = 5%, then `P-value = 0.8246` > *alpha*, and we can't reject the null hypothesis.This value tell us that Ho is the best option, therefore Pnew<Pold***

----

## Part III - Regression
In this final part, we will see that the result we acheived in the previous A/B test can also be acheived by performing regression.<br>
Since each row is either a conversion or no conversion, the **logistic regression** should be perform in this case.<br>    
The goal here is to use **statsmodels** to fit the regression model(using the `Logit` function).
<br>
To see if there is a significant difference in conversion based on which page a customer receives. However, we create a column for the intercept, and create a dummy<br>
variable column for which page each user received. Was added an **intercept** column, as well as an **ab_page** column, which is **1** when an individual receives <br>
the treatment and **0** if control.
<br>
After fit the model with this columns we obtain this summary table:<br>

|  variable   |    coef	  |  std err  |      z     |     P>z      |  [0.025  |  0.975]  |
|-------------|-----------|-----------|------------|--------------|----------|----------|
|  intercept  |  -1.9888  |  0.008    |  -246.669  |     0.000    | -2.005   | -1.973   | 
|  ab_page    |  -0.0150  |  0.011    |  -1.311    |   **0.190**  | -0.037   |  0.007   |
<br>

**P-value** found for **ab_page** is 0.19 and it represents the measure of how compatible our data are with the null hypothesis. The p-values are different <br>
because in part2 the null and alternative hypothesis are diferrent of part3, there in part2 we have a *unicaudal statistic* and here we have a *bicaudal statistic*.<br> 
That is why the values differ for each other in part2 and part3.
<br>
**In part2, Ho : Pnew<=Pold and H1: Pnew > Pold <br>
In part3, Ho : there is no difference between Pnew and Pold and H1: Pnew and Pold are diferrent.** 
<br>

Now along with testing if the conversion rate changes for different pages, also was added an effect based on which country a user lives.<br>
We use `countries.csv` dataset and merge together our datasets on the approporiate rows, and aftera few steps we get this sample table:<br>


|    user_id    |         timestamp	          |     group       |  landing_page  | converted |  intercept | ab_page | UK | US |
|---------------|-----------------------------|-----------------|----------------|-----------|------------|---------|----|----| 
|     834778    | 2017-01-14 23:08:43.3049987 |     control     |    old_page    |     0     |      1     |    0    |  1 |  0 |
<br>
We have 3 countries : 'UK','US' and 'CA', but 'CA' was hidden by logical criteria.  When 'UK' and 'US' are **0** then 'CA' is **1** .
<br>
Now,we would like to look at an interaction between page and country to see if there significant effects on conversion.<br>

After the `Logit` model is trained with the new features, we obtain this:<br>


|  variable   |    coef	  |  std err  |      z     |     P>z      |  [0.025  |  0.975]  |
|-------------|-----------|-----------|------------|--------------|----------|----------|
|  intercept  |  -2.0300  |   0.027   |  -76.249   |     0.000    | -2.082	 | -1.978   | 
|  ab_page    |  -0.0149  |   0.011   |   -1.307   |     0.191    | -0.037   |  0.007   |
|     UK      |   0.0506  |   0.028	  |   1.784    |     0.074    | -0.005   |  0.106   |  
|     US      |  0.0408   |   0.027	  |   1.516	   |    0.130	  | -0.012	 |  0.093   |
<br>

Here, we can analyze two things.<br>
*First* : The **p-values** (except the intercept)are greater than **0.05**, which show us that we cant reject the null hypothesis(Pnew=Pold)<br> 
*Second* : How the country variables have small **p-values**, but still are greater than **0.05**,it means that we could not reject the summary <br>
function hypothesis that b1,b2,b3...bn = 0. That is, at least the country variables coeficients are close to zero, and it means that this<br> 
variables dont impact the convert decision.
<br>

Finally we will split our data into training and test data to fit our `LogisticRegression` classifier.<br>
After fitting our model, we made predictions to see how well our results were using the `accuracy_score` function from *sklearn*.<br>
Our result was very satisfactory, considering that we did not tuning parameters: **0.8804** <br>

----
### Conclusion
***The various methods used above, such as A/B test and logistic regression, shows us that Pnew <= Pold. It means that the proportion of converted<br> 
users on treatment group was insignificant, that is, the changes in the website from the new page was not significant in terms of conversion rate.***






