---
title: "Experimental Designs"
author: "Jason G. Wallace"
date: "2020-09-29"
output: html_document
---




# Experimental Designs
These are the various designs we have covered in class, with quick notes about how to create them and how to analyze them. This is meant to be a quick cheat-sheet; for a full explanation, see the lecture notes. Most of these designs can be created using the [agricolae](https://cran.r-project.org/web/packages/agricolae) package. The output from agricolae's design.____() family of functions has a lot of useful information, but the part you're probably most interested in are the \$book and \$sketch parts, which have a data table and field layout of the experiment. (Not all designs generate a \$sketch component.)

**Note on randomization:** Remember to use set.seed() to set your random seed before randomizing your design. This will let you recreate it in case something ever goes wrong. Just be sure to use a different random seed if you need a different randomization (e.g., doing the same experiment the next year.)

## Factorial designs
Factorial designs aren't really a separate design, since 'factorial' just means you're making all possible combinations of two (or more) treatments. As such, any design can be a factorial design. Just make sure you put all your treatments (and any interactions you're interested in) into your models.

## Completely randomized design
CRDs are the simplest design where you just have replicates of your treatments arranged randomly in your experiment. 

You can make a CRD in basic R using the rep() and sample() functions to replicate and randomize your samples, respectively.

```r
> treatments=LETTERS[1:10] # Define treatments
> design = rep(treatments, 3) # 3 replicates of each treatment
> design = sample(design, replace=F) # By default, sample() will make a sample the same size as what you give it
> design
##  [1] "B" "A" "D" "D" "G" "J" "I" "C" "A" "I" "J" "F" "D" "H" "J" "H" "B" "F" "I"
## [20] "F" "B" "E" "G" "E" "E" "A" "C" "H" "C" "G"
```

You can also use design.crd() from agricolae to do the same thing

```r
> library(agricolae)
> treatments=LETTERS[1:10] # Define treatments
> design = design.crd(trt=treatments, r=3) # 3 replicates of each treatment
> head(design$book) # Fieldbook data (only first few lines shown)
##   plots r treatments
## 1   101 1          D
## 2   102 1          A
## 3   103 2          A
## 4   104 1          I
## 5   105 1          E
## 6   106 1          H
```

Since CRDs only involve your treatments, there are no special considerations to analyze them by [ANOVA] or [Linear Regression].


```r
> berries=read.csv('data/blueberries.csv')
> mymodel = lm(resistance ~ variety, data=berries)
> anova(mymodel)
## Analysis of Variance Table
## 
## Response: resistance
##           Df Sum Sq Mean Sq F value  Pr(>F)   
## variety    5  45.99  9.1980  4.4804 0.00503 **
## Residuals 24  49.27  2.0529                   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```



## Randomized complete block designs
RCBDs are the workhorses of experimental design. They're probably the most common overall, and you should probably start with them as your default design and change to something else only if there's a reason RCBD won't work.

While you *can* make your own RCBD using basic R, it's much easier to let agricolae do it for you

```r
> library(agricolae)
> treatments=LETTERS[1:10] # Define treatments
> design = design.rcbd(trt=treatments, r=3) # 3 replicates of each treatment
> design$sketch # Field layout
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,] "J"  "H"  "F"  "B"  "A"  "D"  "I"  "G"  "E"  "C"  
## [2,] "C"  "D"  "E"  "J"  "B"  "F"  "H"  "A"  "I"  "G"  
## [3,] "H"  "I"  "C"  "D"  "E"  "B"  "G"  "A"  "F"  "J"
> head(design$book) # Fieldbook data (only first few lines shown)
##   plots block treatments
## 1   101     1          J
## 2   102     1          H
## 3   103     1          F
## 4   104     1          B
## 5   105     1          A
## 6   106     1          D
```

When analyzing RCBDs, make sure to include your block term. If you're using a fixed-effects model (meaning **lm()**), blocks should go first in your model because they're a nuisance variable. If using a mixed-effects model, blocks are usually treated as random effects because we don't care about the specific blocks you used.


```r
> # Factorial analysis of bananas
> bananas = read.csv('data/bananas.csv')
> 
> # Fixed effects model
> fixed = lm(resistance ~ block + variety * fungicide, data=bananas)
> anova(fixed)
## Analysis of Variance Table
## 
## Response: resistance
##                   Df  Sum Sq Mean Sq F value    Pr(>F)    
## block              5  9.5397  1.9079  6.7502 3.462e-05 ***
## variety            4 16.1644  4.0411 14.2973 1.401e-08 ***
## fungicide          2  2.4407  1.2203  4.3175   0.01706 *  
## variety:fungicide  8  4.6349  0.5794  2.0498   0.05276 .  
## Residuals         70 19.7853  0.2826                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> # Mixed effects model
> library(lmerTest)
## Loading required package: lme4
## Loading required package: Matrix
## 
## Attaching package: 'lmerTest'
## The following object is masked from 'package:lme4':
## 
##     lmer
## The following object is masked from 'package:stats':
## 
##     step
> mixed = lmer(resistance ~ (1|block) + variety * fungicide, data=bananas)
> 
> # ANOVA of fixed effects
> anova(mixed) 
## Type III Analysis of Variance Table with Satterthwaite's method
##                    Sum Sq Mean Sq NumDF DenDF F value    Pr(>F)    
## variety           16.1644  4.0411     4    70 14.2973 1.401e-08 ***
## fungicide          2.4407  1.2203     2    70  4.3175   0.01706 *  
## variety:fungicide  4.6349  0.5794     8    70  2.0498   0.05276 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> # Similar analysis of random effects
> ranova(mixed) 
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## resistance ~ variety + fungicide + (1 | block) + variety:fungicide
##             npar  logLik    AIC   LRT Df Pr(>Chisq)    
## <none>        17 -77.249 188.50                        
## (1 | block)   16 -84.644 201.29 14.79  1  0.0001202 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## Latin Squares
Latin squares involve comparing several treatments that all have the same number of levels. While you can do this on your own, it's much easier to let agricolae do it for you.


```r
> library(agricolae)
> treatments=LETTERS[1:4]
> design=design.lsd(trt=treatments) # Only have to supply the treatment levels
> design$sketch # Field layout
##      [,1] [,2] [,3] [,4]
## [1,] "B"  "C"  "D"  "A" 
## [2,] "C"  "D"  "A"  "B" 
## [3,] "A"  "B"  "C"  "D" 
## [4,] "D"  "A"  "B"  "C"
> head(design$book) # Fieldbook data (only first few lines shown)
##   plots row col treatments
## 1   101   1   1          B
## 2   102   1   2          C
## 3   103   1   3          D
## 4   104   1   4          A
## 5   201   2   1          C
## 6   202   2   2          D
```

Since pairwise interactions only occur once in a Latin Square, you can only test main effects, not interactions


```r
> clover = read.csv("data/clover.csv")
> mymodel = lm(biomass ~ location + scorer + variety, data=clover)
> anova(mymodel)
## Analysis of Variance Table
## 
## Response: biomass
##           Df Sum Sq Mean Sq F value    Pr(>F)    
## location   4  46.86  11.716  6.1434  0.006277 ** 
## scorer     4  19.41   4.853  2.5450  0.094148 .  
## variety    4 348.51  87.127 45.6870 3.629e-07 ***
## Residuals 12  22.88   1.907                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## Graeco-Latin Squares
Graeco-Latin squares are just Latin Squares with an extra main effect added on, so that all pairwise combinations still occur once. Although in theory you can layer these many layers deep (with 6, 7, 8, etc. effects you're testing), in practice you really don't want to go deeper than four (row, column, treatment 1, treatment 2). Probably for this reason, agricolae only does the most basic Graeco-Latin Square:



```r
> library(agricolae)
> treat1=LETTERS[1:4]
> treat2=LETTERS[5:8]
> design=design.graeco(trt1=treat1, trt2=treat2) 
> design$sketch # Field layout
##      [,1]  [,2]  [,3]  [,4] 
## [1,] "D F" "C E" "A H" "B G"
## [2,] "C H" "D G" "B F" "A E"
## [3,] "A G" "B H" "D E" "C F"
## [4,] "B E" "A F" "C G" "D H"
> head(design$book) # Fieldbook data (only first few lines shown)
##   plots row col treat1 treat2
## 1   101   1   1      D      F
## 2   102   1   2      C      E
## 3   103   1   3      A      H
## 4   104   1   4      B      G
## 5   201   2   1      C      H
## 6   202   2   2      D      G
```

Analysis of Graeco-Latin squares is identical to Latin Squares, with the only real constraint being that you can only test main effects, not interactions.

## Balanced Incomplete Block Designs
BIBD can be easily constructed in agricolae, assuming the parameters you've chosen are compatible. (Not all combinations of treatment levels, reps, and block size can actually give a working BIBD.)

```r
> # Make a basic BIBD
> library(agricolae)
> treatments = c("A","B","C","D")
> bibd = design.bib(trt=treatments, k=3) # R will find smallest # of reps that works
## 
## Parameters BIB
## ==============
## Lambda     : 2
## treatmeans : 4
## Block size : 3
## Blocks     : 4
## Replication: 3 
## 
## Efficiency factor 0.8888889 
## 
## <<< Book >>>
> bibd$sketch # Check layout
##      [,1] [,2] [,3]
## [1,] "A"  "D"  "C" 
## [2,] "C"  "A"  "B" 
## [3,] "A"  "B"  "D" 
## [4,] "D"  "C"  "B"
> bibd$book # Fieldbook table
##    plots block treatments
## 1    101     1          A
## 2    102     1          D
## 3    103     1          C
## 4    201     2          C
## 5    202     2          A
## 6    203     2          B
## 7    301     3          A
## 8    302     3          B
## 9    303     3          D
## 10   401     4          D
## 11   402     4          C
## 12   403     4          B
> 
> 
> # Wrong number of reps
> design.bib(trt=LETTERS[1:8], k=4, r=3)
## 
##  Change r by  7, 14 ...
> # Poor value of k
> design.bib(trt=LETTERS[1:8], k=6, r=3)
## Other k <>  6 ; 1<k< 8
> # Just plain error
> design.bib(trt=LETTERS[1:8], k=5)
## Error in rep(k, b): invalid 'times' argument
```

Analyzing a BIBD is pretty straightforward to analyze, since it's just an experiment parcelled out into incomplete blocks.

```r
> onions = read.csv("data/onions.csv")
> mymodel = lm(disease ~ block + variety, data=onions)
> anova(mymodel)
## Analysis of Variance Table
## 
## Response: disease
##           Df Sum Sq Mean Sq F value  Pr(>F)  
## block      9 7.1787 0.79763  2.1811 0.08318 .
## variety    4 4.5489 1.13722  3.1098 0.04516 *
## Residuals 16 5.8511 0.36569                  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## Confounded designs
Building a confounded design requires a little work with the [conf.design](https://cran.r-project.org/package=conf.design) package. It's kind of tricky because you have to specify which things are confounded, then do the replication and randomization yourself. You won't need to make your own confounded designs in this class for that reason.

*Analyzing* a confounded design, however, is straightforward and follows the same pattern as RCBD, BIBD, Latin Squares, etc. Just be aware that you won't be able to test whatever effect(s) you confounded (unless it's only a partially confounded design).


```r
> mydata = read.csv("data/confounded.csv")
> mymodel = lm(disease ~ covercrop * solarize, data=mydata)
> anova(mymodel)
## Analysis of Variance Table
## 
## Response: disease
##                    Df Sum Sq Mean Sq F value  Pr(>F)  
## covercrop           1 14.098  14.098  2.1248 0.21867  
## solarize            1 79.758  79.758 12.0209 0.02565 *
## covercrop:solarize  1 46.465  46.465  7.0030 0.05720 .
## Residuals           4 26.540   6.635                  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## Split-plot designs
Split-plot designs involve using one of your main effects as a blocking factor, usually because it's hard to change. Split-plots can be generated in agricolae by specifying the two treatments, number of reps, and the design for the whole plots

```r
> library(agricolae)
> wholeplot=c("A","B","C")
> subplot=c("test1", "test2", "test3","test4", "test5")
> # Make a design with whole plots in RCBD with 3 reps
> mydesign = design.split(trt1=wholeplot, trt2=subplot, r=3, design='rcbd')
> head(mydesign$book)
##   plots splots block wholeplot subplot
## 1   101      1     1         A   test1
## 2   101      2     1         A   test2
## 3   101      3     1         A   test4
## 4   101      4     1         A   test3
## 5   101      5     1         A   test5
## 6   102      1     1         C   test1
```

Since all the measurements within each whole-plot are correlated with each other, we need to account for that in our analyses. Although you can specify additional error terms with [aov()], it's easier to just use [random effects and lme4] with lmer().

**Note:** Make sure each unique whole-plot has a unique label so that they are treated correctly.


```r
> library(lmerTest)
> mydata = read.csv('data/splitplot.csv') # CRD, so no block effect
> mymodel = lmer(yield ~ fertilizer * variety + (1|wholeplot), data=mydata)
## boundary (singular) fit: see ?isSingular
> anova(mymodel) # Fixed effect ANOVA
## Type III Analysis of Variance Table with Satterthwaite's method
##                    Sum Sq Mean Sq NumDF DenDF  F value    Pr(>F)    
## fertilizer         452.40  452.40     1    16 343.5962 3.078e-12 ***
## variety            159.38   53.13     3    16  40.3494 1.091e-07 ***
## fertilizer:variety  17.13    5.71     3    16   4.3354    0.0204 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> ranova(mymodel) # Random effect test (similar to ANOVA)
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## yield ~ fertilizer + variety + (1 | wholeplot) + fertilizer:variety
##                 npar  logLik    AIC         LRT Df Pr(>Chisq)
## <none>            10 -29.298 78.597                          
## (1 | wholeplot)    9 -29.298 76.597 -7.1054e-15  1          1
```

If the whole-plots are in a RCBD design, make sure to include the blocks in the model as well.


```r
> library(lmerTest)
> mydata = read.csv('data/splitplot_rcbd.csv') 
> mymodel = lmer(yield ~ fertilizer * variety + (1|block) + (1|wholeplot), data=mydata)
> anova(mymodel) # Fixed effect ANOVA
## Type III Analysis of Variance Table with Satterthwaite's method
##                    Sum Sq Mean Sq NumDF DenDF  F value    Pr(>F)    
## fertilizer         224.97 112.487     2     4 104.1275 0.0003551 ***
## variety            500.88 125.221     4    24 115.9153 2.505e-15 ***
## fertilizer:variety  43.67   5.459     8    24   5.0536 0.0009221 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> ranova(mymodel) # Random effect test (similar to ANOVA)
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## yield ~ fertilizer + variety + (1 | block) + (1 | wholeplot) + 
##     fertilizer:variety
##                 npar  logLik    AIC    LRT Df Pr(>Chisq)  
## <none>            18 -56.944 149.89                       
## (1 | block)       17 -59.320 152.64 4.7522  1    0.02926 *
## (1 | wholeplot)   17 -57.570 149.14 1.2508  1    0.26340  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

## Strip-plot designs

Strip-plots are almost identical to split-splots, except you have two whole-plot random effects (one for each strip), plus another one for blocks if they are arranged in RCBD (which they usually are).


```r
> library(lmerTest)
> mydata = read.csv('data/strip_plot.csv') 
> mymodel = lmer(disease ~ fertilizer * irrigation + (1|block) + (1|strip1) + (1|strip2), data=mydata)
## boundary (singular) fit: see ?isSingular
> anova(mymodel) # Fixed effect ANOVA
## Type III Analysis of Variance Table with Satterthwaite's method
##                        Sum Sq Mean Sq NumDF   DenDF F value  Pr(>F)  
## fertilizer            118.827  59.414     2  6.8856  6.9135 0.02256 *
## irrigation             43.898  21.949     2  7.5786  2.5541 0.14194  
## fertilizer:irrigation  87.991  21.998     4 11.8864  2.5597 0.09341 .
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
> 
> ranova(mymodel) # Random effect test (similar to ANOVA)
## ANOVA-like table for random-effects: Single term deletions
## 
## Model:
## disease ~ fertilizer + irrigation + (1 | block) + (1 | strip1) + 
##     (1 | strip2) + fertilizer:irrigation
##              npar  logLik    AIC    LRT Df Pr(>Chisq)  
## <none>         13 -82.829 191.66                       
## (1 | block)    12 -82.829 189.66 0.0000  1    1.00000  
## (1 | strip1)   12 -83.864 191.73 2.0714  1    0.15008  
## (1 | strip2)   12 -84.804 193.61 3.9517  1    0.04682 *
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

## Augmented designs

