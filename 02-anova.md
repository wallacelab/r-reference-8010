---
title: "ANOVA"
author: "Jason G. Wallace"
date: "2020-09-29"
output: html_document
---




# ANOVA
ANOVA (ANalysis Of VAriance) is one method we use to determine how much of our results are due to different treatments. R actually has two functions to do ANOVA: 

* [aov()](#aov) is basic ANOVA and will be used here. 
* [anova()](linear_regression.nb.html#anova) and [Anova()](linear_regression.nb.html#anova2) are used in conjunction with [linear regression](linear_regression.nb.html) and so is discussed on that page instead.



## aov() {#aov}
The aov() function performs direct ANOVA analysis. It is easiest to have your data in a [data frame](r-basics.nb.html#data_frames), with your variables as column names. (See [this document](http://faculty.chicagobooth.edu/richard.hahn/teaching/formulanotation.pdf) for an explanation of R's formula operators.)

```r
> # Example ANOVA of maize yield data
> mydata = read.csv('data/maize.csv')
> mymodel = aov(yield ~ block + variety, data=mydata)
> mymodel
## Call:
##    aov(formula = yield ~ block + variety, data = mydata)
## 
## Terms:
##                   block variety Residuals
## Sum of Squares  29.9400 12.5425    9.9125
## Deg. of Freedom       7       3        21
## 
## Residual standard error: 0.6870399
## Estimated effects may be unbalanced
```

The basic output of aov() gives you the sums of squares and DOF for each of your effects, but does not calculate the F-statistics and p-values we need to determine significance. For that you need to call summary() on the aov() results:

```r
> # 'mymodel' is the result of the call to aov(), above
> mydata = read.csv('data/maize.csv')
> mymodel = aov(yield ~ block + variety, data=mydata)
> summary(mymodel)
##             Df Sum Sq Mean Sq F value   Pr(>F)    
## block        7 29.940   4.277   9.061 3.74e-05 ***
## variety      3 12.543   4.181   8.857 0.000546 ***
## Residuals   21  9.912   0.472                     
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

## anova()
anova() is a generic function to perform ANOVA analysis on some sort of fitted model, such as the output from lm(). Its output is very similar to that of aov(), but we prefer using anova() because linear regression is more versatile. anova() can only do Type I sums of squares, so model order matters.

```r
> mydata = read.csv('data/maize.csv')
> mymodel = lm(yield ~ block + variety, data=mydata)
> anova(mymodel)
## Analysis of Variance Table
## 
## Response: yield
##           Df  Sum Sq Mean Sq F value    Pr(>F)    
## block      7 29.9400  4.2771  9.0613 3.743e-05 ***
## variety    3 12.5425  4.1808  8.8573 0.0005458 ***
## Residuals 21  9.9125  0.4720                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

See the section on [lmerTest] for how to perform ANOVA on random-effect and mixed-effect linear models.


## Anova()
Anova() is part of the [car](https://cran.r-project.org/web/packages/car/index.html) package, and we use it to do Type II and Type III sums-of-squares. Note that you have to change the way R handles contrasts for Type III to run correctly.

```r
> # Read in data and fit mdoel
> library(car)
> bananas = read.csv('data/bananas.csv')
> mymodel = lm(resistance ~ variety * fungicide, data=bananas)
> 
> # Type II
> Anova(mymodel, type=2)
## Anova Table (Type II tests)
## 
## Response: resistance
##                    Sum Sq Df F value    Pr(>F)    
## variety           16.1644  4 10.3353 1.014e-06 ***
## fungicide          2.4407  2  3.1211   0.04989 *  
## variety:fungicide  4.6349  8  1.4817   0.17822    
## Residuals         29.3250 75                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> # Type III
> options(contrasts = c("contr.sum", "contr.poly")) # So Type III works right
> Anova(mymodel, type=3)
## Anova Table (Type III tests)
## 
## Response: resistance
##                    Sum Sq Df  F value    Pr(>F)    
## (Intercept)       101.682  1 260.0554 < 2.2e-16 ***
## variety             7.297  4   4.6654  0.002038 ** 
## fungicide           2.830  2   3.6189  0.031594 *  
## variety:fungicide   4.635  8   1.4817  0.178217    
## Residuals          29.325 75                       
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
 

## Tukey tests {#tukey_tests}

ANOVA can tell you that there is a significant difference among your factor levels somwhere, but cannot tell you where. For that, you need a Tukey Test (Tukey's Honest Significant Difference test). This is basically a bunch of pairwise t-tests to determine which pair(s) of factor levels are different from each other, with correction for the number of comparisons you're making. A Tukey test should _only_ be run if your ANOVA indicates there is a significant difference somewhere.

There are two methods to do Tukey tests in R. The default TukeyHSD() takes a result from aov() and gives you all pairwise p-values for each factor level. (Note: it does not work on linear regression output from lm():

```r
> mydata = read.csv('data/maize.csv')
> mymodel = aov(yield ~ block + variety, data=mydata)
> TukeyHSD(mymodel)
##   Tukey multiple comparisons of means
##     95% family-wise confidence level
## 
## Fit: aov(formula = yield ~ block + variety, data = mydata)
## 
## $block
##                 diff         lwr         upr     p adj
## block2-block1  0.850 -0.77947909  2.47947909 0.6574659
## block3-block1 -1.050 -2.67947909  0.57947909 0.4113645
## block4-block1 -2.000 -3.62947909 -0.37052091 0.0095724
## block5-block1 -0.225 -1.85447909  1.40447909 0.9997193
## block6-block1  1.325 -0.30447909  2.95447909 0.1685496
## block7-block1  0.050 -1.57947909  1.67947909 1.0000000
## block8-block1 -0.250 -1.87947909  1.37947909 0.9994400
## block3-block2 -1.900 -3.52947909 -0.27052091 0.0151276
## block4-block2 -2.850 -4.47947909 -1.22052091 0.0001830
## block5-block2 -1.075 -2.70447909  0.55447909 0.3834069
## block6-block2  0.475 -1.15447909  2.10447909 0.9728357
## block7-block2 -0.800 -2.42947909  0.82947909 0.7186047
## block8-block2 -1.100 -2.72947909  0.52947909 0.3565008
## block4-block3 -0.950 -2.57947909  0.67947909 0.5315240
## block5-block3  0.825 -0.80447909  2.45447909 0.6883738
## block6-block3  2.375  0.74552091  4.00447909 0.0016661
## block7-block3  1.100 -0.52947909  2.72947909 0.3565008
## block8-block3  0.800 -0.82947909  2.42947909 0.7186047
## block5-block4  1.775  0.14552091  3.40447909 0.0265304
## block6-block4  3.325  1.69552091  4.95447909 0.0000217
## block7-block4  2.050  0.42052091  3.67947909 0.0075991
## block8-block4  1.750  0.12052091  3.37947909 0.0296337
## block6-block5  1.550 -0.07947909  3.17947909 0.0698250
## block7-block5  0.275 -1.35447909  1.90447909 0.9989637
## block8-block5 -0.025 -1.65447909  1.60447909 1.0000000
## block7-block6 -1.275 -2.90447909  0.35447909 0.2017310
## block8-block6 -1.575 -3.20447909  0.05447909 0.0629293
## block8-block7 -0.300 -1.92947909  1.32947909 0.9981982
## 
## $variety
##                diff          lwr       upr     p adj
## DK884-DK417  0.0625 -0.895002939 1.0200029 0.9977921
## M154-DK417  -0.7375 -1.695002939 0.2200029 0.1711460
## PH991-DK417  1.0250  0.067497061 1.9825029 0.0330723
## M154-DK884  -0.8000 -1.757502939 0.1575029 0.1232938
## PH991-DK884  0.9625  0.004997061 1.9200029 0.0485161
## PH991-M154   1.7625  0.804997061 2.7200029 0.0002401
```


More useful output comes from the HSD.test() function in the agricolae package, which will tell you the groupings for a factor. It can also work on the output from lm(). (Note that in boht cases you have to specify which factor you're interested in):

```r
> library(agricolae)
> mydata = read.csv('data/maize.csv')
> mymodel = aov(yield ~ block + variety, data=mydata)
> tukey = HSD.test(mymodel, trt='variety') # the 'trt' argument specified which factor to compare
> tukey # HDS.test() doesn't print out the result by default, so have to print it out manually
## $statistics
##     MSerror Df   Mean       CV       MSD
##   0.4720238 21 6.7375 10.19725 0.9575029
## 
## $parameters
##    test  name.t ntr StudentizedRange alpha
##   Tukey variety   4         3.941878  0.05
## 
## $means
##        yield       std r Min Max   Q25  Q50   Q75
## DK417 6.6500 1.1928358 8 4.9 8.5 5.725 6.85 7.250
## DK884 6.7125 1.2540648 8 4.8 9.1 6.000 6.65 7.175
## M154  5.9125 0.7567553 8 4.5 6.6 5.375 6.30 6.425
## PH991 7.6750 1.4577380 8 5.4 9.6 6.850 8.10 8.450
## 
## $comparison
## NULL
## 
## $groups
##        yield groups
## PH991 7.6750      a
## DK884 6.7125      b
## DK417 6.6500      b
## M154  5.9125      b
## 
## attr(,"class")
## [1] "group"
> 
> # HSD.test() also works with linear regression output
> mylm = lm(yield ~ block + variety, data=mydata)
> HSD.test(mymodel, trt='variety')$group
##        yield groups
## PH991 7.6750      a
## DK884 6.7125      b
## DK417 6.6500      b
## M154  5.9125      b
```

Although there is a lot of information in the output from HSD.test(), the $groups section is what we usually care about, since that tells us which factor levels are/are not significantly different from each other.


[Index](index.nb.html)
