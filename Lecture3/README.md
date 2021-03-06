STAT406 - Lecture 3 notes
================
Matias Salibian-Barrera
2017-09-15

Lecture slides
--------------

-   A preliminary version of the lecture slides is [here](STAT406-17-lecture-3.pdf).
-   The activity hand-out is [here](lecture3-activity.pdf).

Cross-validation when the model is chosen using the data
--------------------------------------------------------

In this document we study how to perform cross-validation when the model was selected or determined using the training data. Consider the following synthetic data set

``` r
dat <- read.table('fallacy.dat', header=TRUE, sep=',')
```

This is the same data used in class. In this example we know what the "true" model is, and thus we also know what the "optimal" predictor is. However, let us ignore this knowledge, and build a linear model instead. Given how many variables are available, we use forward stepwise (AIC-based) to select a good subset of them to include in our linear model:

``` r
library(MASS)
p <- ncol(dat)
null <- lm(Y~1, data=dat)
full <- lm(Y~., data=dat) # needed for stepwise
step.lm <- stepAIC(null, scope=list(lower=null, upper=full), trace=FALSE)
```

Without thinking too much, we use 50 runs of 5-fold CV (ten runs) to compare the MSPE of the **null** model (which we know is "true") and the one we obtained using forward stepwise:

``` r
n <- nrow(dat)
ii <- (1:n) %% 5 + 1
set.seed(17)
N <- 50
mspe.n <- mspe.st <- rep(0, N)
for(i in 1:N) {
  ii <- sample(ii)
  pr.n <- pr.st <- rep(0, n)
  for(j in 1:5) {
    tmp.st <- update(step.lm, data=dat[ii != j, ])
    pr.st[ ii == j ] <- predict(tmp.st, newdata=dat[ii == j, ])
    pr.n[ ii == j ] <- with(dat[ii != j, ], mean(Y))
  }
  mspe.st[i] <- with(dat, mean( (Y - pr.st)^2 ))
  mspe.n[i] <- with(dat, mean( (Y - pr.n)^2 ))
}
boxplot(mspe.st, mspe.n, names=c('Stepwise', 'NULL'), col=c('gray60', 'hotpink'), main='Wrong')
```

![](README_files/figure-markdown_github-ascii_identifiers/wrong-1.png)

``` r
summary(mspe.st)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##  0.5988  0.6528  0.6643  0.6745  0.7033  0.7682

``` r
summary(mspe.n)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   1.043   1.049   1.055   1.057   1.061   1.087

-   **Something is wrong!** What? Why?
-   What would you change above to obtain reliable estimates for the MSPE of the model selected with the stepwise approach?

<!-- ## Correlated covariates -->
<!-- Technological advances in recent decades have resulted in data  -->
<!-- being collected in a fundamentally different way from the way  -->
<!-- it was when "classical" statistical methods were proposed.  -->
<!-- Specifically, it is not at all uncommon to have data sets with -->
<!-- an abundance of potentially useful explanatory variables.  -->
<!-- Sometimes the investigators are not sure which of them can be  -->
<!-- expected to be useful or meaningful. In many applications one -->
<!-- finds data with many more variables than cases.  -->
<!-- A consequence of this "wide net" data collection strategy is  -->
<!-- that many of the explanatory variables may be correlated with -->
<!-- each other. In what follows we will illustrate some of the -->
<!-- problems that this can cause both when training and interpreting -->
<!-- models, and also with the resulting predictions. -->
<!-- ### Significant variables "dissappear" -->
<!-- Consider the air pollution data set, and the fit to the  -->
<!-- **reduced** linear regression model used previously in class: -->
<!-- ```{r signif} -->
<!-- # Correlated covariates -->
<!-- x <- read.table('../Lecture1/rutgers-lib-30861_CSV-1.csv', header=TRUE, sep=',') -->
<!-- reduced <- lm(MORT ~ POOR + HC + NOX + HOUS + NONW, data=x) -->
<!-- round( summary(reduced)$coef, 3) -->
<!-- ``` -->
<!-- Note that all coefficients seem to be significant based on -->
<!-- the individual tests of hypothesis (with `POOR` and  -->
<!-- `HOUS` maybe only marginally so). In this sense all 5 -->
<!-- explanatory varibles in this model appear to be relevant. -->
<!-- Now, we fit the **full** model, that is, we include -->
<!-- all available explanatory variables in the data set: -->
<!-- ```{r signif2} -->
<!-- full <- lm(MORT ~ ., data=x) -->
<!-- round( summary(full)$coef, 3) -->
<!-- ``` -->
<!-- Now we have many more parameters to estimate, and while two of -->
<!-- them appear to be significantly different from zero (`NONW` -->
<!-- and `PREC`), all the others seem to be redundant.  -->
<!-- In particular, note that the p-values for the individual -->
<!-- test of hypotheses for 4 out of the 5   -->
<!-- regression coefficients for the variables of the **reduced** -->
<!-- model have now become not significant. -->
<!-- ```{r signif3} -->
<!-- round( summary(full)$coef[ names(coef(reduced)), ], 3) -->
<!-- ``` -->
<!-- ### Why does this happen?  -->
<!-- Recall that the covariance matrix of the least squares estimator involves the -->
<!-- inverse of (X'X), where X' denotes the transpose of the n x p matrix X (that -->
<!-- contains each vector of explanatory variables as a row). It is easy to see  -->
<!-- that if two columns of X are linearly dependent, then X'X will be rank deficient.  -->
<!-- When two columns of X are "close" to being linearly dependent (e.g. their -->
<!-- linear corrleation is high), then the matrix X'X will be ill-conditioned, and -->
<!-- its inverse will have very large entries. This means that the estimated  -->
<!-- standard errors of the least squares estimator will be unduly large, resulting -->
<!-- in non-significant test of hypotheses for each parameter separately, even if -->
<!-- the global test for all of them simultaneously is highly significant. -->
<!-- ### Why is this a problem if we are interested in prediction? -->
<!-- Although in many applications one is interested in interpreting the parameters -->
<!-- of the model, even if one is only trying to fit / train a model to do -->
<!-- predictions, highly variable parameter estimators will typically result in -->
<!-- a noticeable loss of prediction accuracy. This can be easily seen from the  -->
<!-- bias / variance factorization of the mean squared prediction error (MSPE)  -->
<!-- mentioned in class. Hence, better predictions can be obtained if one -->
<!-- uses less-variable parameter estimators.  -->
<!-- ### What can we do? -->
<!-- A commonly used strategy is to remove some explanatory variables from the -->
<!-- model, leaving only non-redundant covariates. However, this is easier said than -->
<!-- done. You have seen some strategies in other courses (stepwise variable selection, etc.) -->
<!-- In coming weeks we will investigate other methods to deal with this problem. -->
