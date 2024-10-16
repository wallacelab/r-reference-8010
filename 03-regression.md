---
title: "Linear Regression"
author: "Jason G. Wallace"
date: "2020-09-29"
output: html_document
---



# Linear Regression
Linear regression is the workhorse of statistics both in and out of this class. 

## lm()
lm() (for "linear model") is the basic linear regression function. Fitting models is the same as fitting them with aov(), and it is recommended to have all your variables in a single data frame with the variable names as columns. (See [this document](http://faculty.chicagobooth.edu/richard.hahn/teaching/formulanotation.pdf) for an explanation of R's formula operators.) The summary() function will give you the model coefficients and statistical tests.

```r
> mydata = read.csv('data/maize.csv')
> mymodel = lm(yield ~ block + variety, data=mydata)
> summary(mymodel)
## 
## Call:
## lm(formula = yield ~ block + variety, data = mydata)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -1.0000 -0.4188  0.1000  0.3750  0.9125 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    6.8125     0.4028  16.912 1.04e-13 ***
## blockblock2    0.8500     0.4858   1.750 0.094779 .  
## blockblock3   -1.0500     0.4858  -2.161 0.042369 *  
## blockblock4   -2.0000     0.4858  -4.117 0.000491 ***
## blockblock5   -0.2250     0.4858  -0.463 0.648025    
## blockblock6    1.3250     0.4858   2.727 0.012618 *  
## blockblock7    0.0500     0.4858   0.103 0.919002    
## blockblock8   -0.2500     0.4858  -0.515 0.612204    
## varietyDK884   0.0625     0.3435   0.182 0.857375    
## varietyM154   -0.7375     0.3435  -2.147 0.043636 *  
## varietyPH991   1.0250     0.3435   2.984 0.007079 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.687 on 21 degrees of freedom
## Multiple R-squared:  0.8108,	Adjusted R-squared:  0.7207 
## F-statistic:     9 on 10 and 21 DF,  p-value: 1.406e-05
```

See the section on [ANOVA] for how to do an ANOVA analysis of lm() results.

## Random effects and lme4
The lme4 package lets us make random effect models with the function lmer() (linear mixed effect regression; often pronounced 'lemur').

Random effects are specified by putting "(1|name)" in the formula, where 'name' is your variable name. (Although you can tweak the random effects significantly by changing what else is inside the parentheses, you don't need to get that advanced for this class.) Random effects are usually put last in the model formula.

```r
> library(lme4)
## Loading required package: Matrix
> mydata = read.csv('data/maize.csv')
> mymodel = lmer(yield ~ variety + (1|block), data=mydata)
> summary(mymodel)
## Linear mixed model fit by REML ['lmerMod']
## Formula: yield ~ variety + (1 | block)
##    Data: mydata
## 
## REML criterion at convergence: 82.2
## 
## Scaled residuals: 
##     Min      1Q  Median      3Q     Max 
## -1.5799 -0.6446  0.1855  0.5195  1.5489 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  block    (Intercept) 0.9513   0.9753  
##  Residual             0.4720   0.6870  
## Number of obs: 32, groups:  block, 8
## 
## Fixed effects:
##              Estimate Std. Error t value
## (Intercept)    6.6500     0.4218  15.766
## varietyDK884   0.0625     0.3435   0.182
## varietyM154   -0.7375     0.3435  -2.147
## varietyPH991   1.0250     0.3435   2.984
## 
## Correlation of Fixed Effects:
##             (Intr) vDK884 vrM154
## varityDK884 -0.407              
## varietyM154 -0.407  0.500       
## varityPH991 -0.407  0.500  0.500
```

### Extracting BLUEs and BLUPs
While coefficients() will get you the BLUEs for a normal lm() model, you want to use fixef() and ranef() to get the fixed (BLUEs) and random (BLUPs) effects of lmer() models

```r
> library(lme4)
> mydata = read.csv('data/maize.csv')
> mymodel = lmer(yield ~ variety + (1|block), data=mydata)
> fixef(mymodel) # Fixed effects (BLUEs)
##  (Intercept) varietyDK884  varietyM154 varietyPH991 
##       6.6500       0.0625      -0.7375       1.0250
> ranef(mymodel) # Random effects (BLUPs)
## $block
##        (Intercept)
## block1  0.14456656
## block2  0.90076090
## block3 -0.78955585
## block4 -1.63471422
## block5 -0.05560252
## block6  1.32334009
## block7  0.18904858
## block8 -0.07784353
## 
## with conditional variances for "block"
```


## lmerTest
Running analyses of lmer() results is much easier with the [lmerTest](https://cran.r-project.org/web/packages/lmerTest/index.html) package installed, since it gives you ANOVA functionality on random effects models.

```r
> library(lmerTest) # Automatically loads lme4
## 
## Attaching package: 'lmerTest'
## The following object is masked from 'package:lme4':
## 
##     lmer
## The following object is masked from 'package:stats':
## 
##     step
> mydata = read.csv('data/maize.csv')
> mymodel = lmer(yield ~ variety + (1|block), data=mydata)
> # ANOVA of fixed effects
> anova(mymodel) 
## Type III Analysis of Variance Table with Satterthwaite's method
##         Sum Sq Mean Sq NumDF DenDF F value    Pr(>F)    
## variety 12.543  4.1808     3    21  8.8573 0.0005458 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> # ANOVA-like analysis of random effects
> ranova(mymodel) 
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## yield ~ variety + (1 | block)
##             npar  logLik     AIC    LRT Df Pr(>Chisq)    
## <none>         6 -41.093  94.186                         
## (1 | block)    5 -48.831 107.662 15.476  1  8.357e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## EM-Means
Estimated Marginal Means are basically the best estimate of the main effect of a model when it's averaged over all levels of other effects. (So, for example, the best estimate of each plant variety averaged over blocks, years, etc.) They are especially useful when some of your treatments don't appear in every block because they help remove the bias that can cause.

EM-Means are calculated with the emmeans package using the emmeans() function.

```r
> library(emmeans) 
> mydata = read.csv('data/maize.csv')
> mymodel = lm(yield ~ block + variety, data=mydata)
> emmeans(mymodel, specs="variety") # Specs specifies which terms you're looking at
##  variety emmean    SE df lower.CL upper.CL
##  DK417     6.65 0.243 21     6.14     7.16
##  DK884     6.71 0.243 21     6.21     7.22
##  M154      5.91 0.243 21     5.41     6.42
##  PH991     7.67 0.243 21     7.17     8.18
## 
## Results are averaged over the levels of: block 
## Confidence level used: 0.95
```

### CLD()

To perform a Tukey-like test and get a compact letter distplay of the results, use CLD()

```r
> library(emmeans) 
> mydata = read.csv('data/maize.csv')
> mymodel = lm(yield ~ block + variety, data=mydata)
> mymeans = emmeans(mymodel, specs="variety")
> CLD(mymeans)
## Warning: 'CLD' will be deprecated. Its use is discouraged.
## See '? CLD' for an explanation. Use 'pwpp' or 'multcomp::cld' instead.
##  variety emmean    SE df lower.CL upper.CL .group
##  M154      5.91 0.243 21     5.41     6.42  1    
##  DK417     6.65 0.243 21     6.14     7.16  1    
##  DK884     6.71 0.243 21     6.21     7.22  1    
##  PH991     7.67 0.243 21     7.17     8.18   2   
## 
## Results are averaged over the levels of: block 
## Confidence level used: 0.95 
## P value adjustment: tukey method for comparing a family of 4 estimates 
## significance level used: alpha = 0.05
```
