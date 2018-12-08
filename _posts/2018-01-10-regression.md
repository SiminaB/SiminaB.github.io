This is of course a bit of an exaggeration! It also depends on the way
things are emphasized in, for example, statistical graduate programs. I
know that my own training program was highly focused on seeing much of
statistics as either a special case or a generalization of some kind of
linear regression model. I kind of love this view and try to emphasize
it whenever I can to collaborators, although I do admit that sometimes
you have to start from special cases in order to build up the whole.

Regression concepts connected to loss functions and conditional means and medians
---------------------------------------------------------------------------------

The term "regression" is currently used in a large variety of ways. In
general, if there a variable *Y*, called the outcome or dependent
variable, and some other variables *X*<sub>1</sub>, *X*<sub>2</sub> etc,
called the explanatory or independent variables, then a regression of
*Y* on the *X*s can be ['any feature of the probability distribution" of
*Y* given (or "conditional on") the
*X*.'](http://www.jstor.org/stable/2727353) These features often relate
to some measure of "average" or "center" of the distribution, such as
the mean or the median.

The simplest case of regression is that of linear regression with a
single explanatory variable. In other words, trying to draw a "best fit
line" through a scatterplot. There are many ways to define the "best
fit" and the one that is usually considered is based on the [quadratic
loss
function](https://en.wikipedia.org/wiki/Loss_function#Quadratic_loss_function)
(also known as squared error loss or L<sub>2</sub> loss.) Basically, if
one seeks to *minimize the sum of the squares of the errors*, i.e. the
differences between the true outcome Y and the fitted values on the line
$\\hat{Y}$, the solution is the "usual" [ordinary least squares
(OLS)](https://en.wikipedia.org/wiki/Ordinary_least_squares). Take this
example where there is a single explanatory variable *X*<sub>1</sub> and
the outcome variable *Y* simply adds some noise. The regression OLS fit
is in blue, as are the segments between fitted values and the true
values of Y. Two other lines are shown in dashed orange.
![](Regression_files/figure-markdown_strict/unnamed-chunk-1-1.png)

The sum of the squared errors for the OLS fit is 6.19, while for the
other two lines it is 7.58 and 10.19.

If there is no explanatory variable, the fitted line is flat and
represents the average (mean) value of the outcome values:
![](Regression_files/figure-markdown_strict/unnamed-chunk-2-1.png)

For the example above, this is clearly a poor fit. In fact, the sum of
the squared errors in this case is 49.34!

In general, the OLS has some nice properties, for example if the noise
terms - defined as
*ϵ* = *Y* − *β*<sub>0</sub> − *β*<sub>1</sub>*X*<sub>1</sub>, where
*β*<sub>0</sub> and *β*<sub>1</sub> are the true intercept and slope of
the line - are independent and identically distributed with finite
variance, the OLS fit is an unbiased estimator of the conditional mean
E(Y|X) (i.e. its mean is equal to the conditional mean) and is in fact
the [minimum-variance unbiased
estimator](https://en.wikipedia.org/wiki/Minimum-variance_unbiased_estimator).
Note that so far I haven't discussed the distribution of Y or even
whether Y is continuous. However, in the case where the noise terms (and
therefore Y conditional on X) are normally distributed, the OLS is also
the maximum likelihood estimator (MLE) of the conditional mean E(Y|X).

One reason why the quadratic loss function is considered very often is
because the mathematics works out very nicely (can solve for the maximum
using the usual calculus-based second derivative test). This also
generalizes nicely if there are more explanatory variables - we can just
include them all in a matrix X and essentially proceed in the same way
with the help of some multivariate calculus. There are however many
other ways of defining the "best fit." However, perhaps one is
interested in estimating an aspect of the distribution of Y|X other than
its mean, such as its median. The median is naturally connected to the
[absolute loss](https://en.wikipedia.org/wiki/Loss_function) (also known
as as the L<sub>1</sub> loss) in the same way that the mean is connected
to the quadratic error loss. In this case, the criterion is to *minimize
the sum of the absolute values of the errors*, also called the [least
absolute deviations (LAD) or least absolute
residuals](https://en.wikipedia.org/wiki/Least_absolute_deviations). In
the absence of an explanatory variable, the flat fitted line represents
the median of the explanatory values. If we look more generally for a
function f where the values f(X) are most similar to Y, with the
similarity being defined using the quadratic loss, then the "best fit"
(which minimizes [the expected prediction error (EPE - see *Elements of
statistical learning*, section
2.4)](https://web.stanford.edu/~hastie/Papers/ESLII.pdf) is the function
f(x) = E(Y|X=x) (conditional mean); by contrast, if it is defined using
the absolute loss, it is f(x) = median(Y|X=x) (conditional median). In
general, medians are more robust than means, given their lack of
dependence on outliers, but the math gets harder much faster due to
discontinuous derivatives which complicate optimization procedures. For
the example above, the results are almost identical whether minimizing
the sum of squares or the sum of absolute values of the errors:

    ##intercept and slope estimated via OLS
    coef(lm(y ~ x1))

    ## (Intercept)          x1 
    ##   0.1263162   0.9645116

    library(quantreg)
    ##intercept and slope estimated via quantile regression with median (see below)
    coef(rq(y ~ x1, tau=0.5))

    ## (Intercept)          x1 
    ##   0.1218339   0.9931817

Similarly, if a [step function (0-1)
loss](http://www.jstor.org/stable/2727353) is used instead, the result
is f(x) = mode(Y|X=x) (conditional mode). Note that the mean, median,
and mode are the most common ways of describing the "center" of a
continuous distribution.

It may also be of interest to consider another quantile besides the
median, which is sometimes the case when considering a variable with an
asymmetric distribution, such as income distributions. [Quantile
regression](https://en.wikipedia.org/wiki/Quantile_regression) considers
different loss functions for this purpose; the loss function used for
the median is however equivalent to the LAD. If the noise terms have an
asymmetric Laplace distribution, the estimator obtained here for a
specific quantile is its
[MLE](http://www.american.edu/cas/economics/info-metrics/pdf/upload/working-paper-bera.pdf).
For the rest of this document, we will generally use "regression" in the
"usual way," considering the square error loss.

More about the OLS fit: Linear algebra and geometric interpretation
-------------------------------------------------------------------

Above we considered the special case of fitting lines in 2 dimensions,
where there was a single explanatory variable and two parameters were
estimated: the slope and the intercept of the "best fitting line." In
general, many variables can be considered. It is common to consider all
these variables in a single matrix *X*, which has each variable as a
column and each sample as a row, with the first column typically
consisting of 1s. The goal is then to estimate a vector of "parameters"
*β*, so that the outcome *Y* is "as close as possible" to *X**β* (thus,
the first element of *β* is the intercept). Under certain linear algebra
assumptions, there is a nice [closed form for the OLS in the
multivariate
case](https://en.wikipedia.org/wiki/Proofs_involving_ordinary_least_squares):
$\\hat{\\beta} = (X'X)^{-1}X'Y$. This means that the fitted values are:
$\\hat{Y}=X\\hat{\\beta} = X(X'X)^{-1}X'Y$. The matrix
*X*(*X*′*X*)<sup>−1</sup>*X*′ is in fact the [projection matrix onto the
space spanned by the columns of
X](https://en.wikipedia.org/wiki/Ordinary_least_squares), which is
another way of conceptualizing that $\\hat{Y}$ represents the closest
vector to *Y* out of all the vectors that are linear combinations of the
explanatory variables. Once again, we define "closest" using the
*L*<sup>2</sup> loss, or, in linear algebra terms, minimize the
*L*<sup>2</sup> norm ||.||: $\\hat{\\beta} = \\arg\\min||Y-X\\beta||$.
It is also interesting to note that solving for *β* in *X**β* = *Y*
would in fact represent an [overdetermined
system](https://en.wikipedia.org/wiki/Ordinary_least_squares) and as
such cannot usually be solved exactly - we thus look for a solutions
that gives a fit that is "close" to Y while at the same time being in
the space spanned by the explanatory variables.

T-tests, ANOVA, and all the rest: Hypothesis testing in linear models
---------------------------------------------------------------------

Perhaps the first thing most people think of when they think statistics
is the t-test. [Student's two-sample
t-test](https://en.wikipedia.org/wiki/Student%27s_t-test#Independent_two-sample_t-test)
is used to compare the means of two distributions, with the null
hypothesis stating that they are equal, for either normally distributed
random variables or for continuous random variables with a large enough
sample size. One of the powerful things about linear regression is that
a two-sample t-test which assumes equal variances is in fact the same as
a linear model where the explanatory variable is a 0/1 variable which
defines group membership:

    set.seed(380148)
    n <- 10
    ##define the two random variables
    ##y1 is from a normal distribution with mean=0, sd=1
    y1 <- rnorm(n,0,1)
    ##y2 is from a normal distribution with mean=0.5, sd=1
    y2 <- rnorm(n,0.5,1)
    ##run a t-test!
    t.test(y1,y2,var.equal = TRUE)

    ## 
    ##  Two Sample t-test
    ## 
    ## data:  y1 and y2
    ## t = -0.66034, df = 18, p-value = 0.5174
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -1.5638050  0.8158546
    ## sample estimates:
    ## mean of x mean of y 
    ## 0.2961889 0.6701641

    ##create a single vector of outcomes and a vector indicating group membership
    y <- c(y1,y2)
    x1 <- c(rep(0,length(y1)),rep(1,length(y2)))
    summary(lm(y~x1))

    ## 
    ## Call:
    ## lm(formula = y ~ x1)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.3130 -0.9469  0.1265  1.0192  1.7432 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)   0.2962     0.4005    0.74    0.469
    ## x1            0.3740     0.5663    0.66    0.517
    ## 
    ## Residual standard error: 1.266 on 18 degrees of freedom
    ## Multiple R-squared:  0.02365,    Adjusted R-squared:  -0.03059 
    ## F-statistic: 0.436 on 1 and 18 DF,  p-value: 0.5174

As we can see, the p-value is the same and the estimated slope is in
fact equal to the differences between means. Since one of the
assumptions behind the properties of the OLS is that the noise terms are
identically distributed, this means that the case of unequal variances
cannot be directly placed in this framework. However, if the variances
are unequal, this means that perhaps other explanatory variables should
be considered, so these could then be included in a regression model.

The same idea holds in the case of ANOVA, where multiple group means are
being compared. One of the benefits of having a regression mindset is
that it very easy to interpret the results in terms of conditional means
and it also allows for a unified framework which can include both
continuous and discrete explanatory variables and interactions between
them, without having to resort to special terminology like two-way
ANOVA, ANCOVA, etc.

In general, we can test whether any linear combination of parameters in
a multivariate model is equal to 0 by performing an
[F-test](https://courses.washington.edu/b515/l6.pdf). This is a powerful
approach, as it includes cases including the equality of two
coefficients and of course, has a special case whether a single
parameter is equal to 0. For this latter case, this approach is
equivalent to using a t-test with *n* − *p* − 1 degrees of freedom,
where *p* is the number of parameters, excluding the intercept. Thus,
once again, the regression approach allows for a more general framework
from which the special cases naturally fall out.

From weighted regression to generalized linear models
-----------------------------------------------------

We mentioned before that one of the properties that is generally assumed
for linear regression is that the noise terms are independent and
identically distributed. In case this does not hold, an [alterative to
OLS](https://en.wikipedia.org/wiki/Generalized_linear_model) is to use
either *weighted least squares* (WLS) or *generalized least squares*
(GLS) approaches. This means that if the variance-covariance matrix of
the noise terms is *W*, then the MLE solution is:
$\\hat{\\beta}=(X'W^{-1}X)^{-1}X'W^{-1}Y$. This is equivalent to
minimizing $\\hat{\\beta} = \\arg\\min(Y-X\\beta)'W^{-1}(Y-X\\beta)$,
known as the *Mahalanobis length,* instead of
$\\hat{\\beta} = \\arg\\min||Y-X\\beta||$ as before. If W has all
off-diagonal terms equal to 0 (i.e. the noise terms are independent),
this is the WLS estimate, otherwise it is the GLS estimate.

What happens if the outcome is not normally distributed? Here we come to
the [generalized linear
model](https://en.wikipedia.org/wiki/Generalized_linear_model). In the
case in which the outcome Y is from a class of distributions known as
the [exponential
family](https://en.wikipedia.org/wiki/Exponential_family) - which
include the normal, exponential, and Bernoulli distributions - instead
of trying to directly estimate the conditional mean *E*(*Y*|*X*) as a
linear function *X**β*, it is generally more convenient to estimate a
transformation of this mean, *g*\[*E*(*Y*|*X*)\]. This is because, for
example, for a 0/1 outcome, using just the OLS will often result in
fitted values outside of the \[0,1\] range, but using a *link function*
g that transforms the (0,1) range corresponding to the probability of
success in a Bernoulli distribution into the entire real line solves
this problem. If the link function is the log-odds (logit), we find
ourselves looking at logistic regression. In the general case, we cannot
obtain a closed form MLE solution, but by applying a first-order Taylor
approximation, we obtain an algorithm that performs [iteratively
reweighted least squares
(IRLS)](https://web.as.uky.edu/statistics/users/pbreheny/760/S13/notes/2-19.pdf),
where the matrix *W* is related to the derivative of the inverse link
function.

Smoothing and regression
------------------------

So far we have discussed fitting linear models between an outcome and
explanatory variables, but what happens if the relationship is clearly
nonlinear? One option is to add more explanatory variables, including
polynomial terms, and fit a linear function to those - for example,
instead of having *X*<sub>1</sub> as an explanatory variable, can
consider *X*<sub>1</sub> and *X*<sub>1</sub><sup>2</sup>, so that we fit
the model
*E*(*Y*|*X*<sub>1</sub>)=*β*<sub>0</sub> + *β*<sub>1</sub>*X*<sub>1</sub> + *β*<sub>2</sub>*X*<sub>1</sub><sup>2</sup>
instead of
*E*(*Y*|*X*<sub>1</sub>)=*β*<sub>0</sub> + *β*<sub>1</sub>*X*<sub>1</sub>.
Note that the function is still linear in the explanatory variables that
are considered. More flexibility can be obtained by using regression
models that have piecewise polynomial terms, known as
[splines](https://en.wikipedia.org/wiki/Spline_(mathematics)). For the
case of a single explanatory variable, another popular approach is
[local regression
(loess)](https://en.wikipedia.org/wiki/Local_regression), which fits
simple linear regression using a moving window, resulting in a smooth
function. Many other advanced approaches are built on these concepts,
including [smoothing
splines](https://en.wikipedia.org/wiki/Smoothing_spline), which place
knots at every observation but have a ["roughness penalty" to reduce
overfitting](https://web.stanford.edu/class/stats202/content/lec17.pdf)
and [generalized additive models
(GAMs)](https://web.stanford.edu/class/stats202/content/lec17.pdf) which
fit smooth functions for each separate explanatory variable.

Regression and machine learning
-------------------------------

*Machine learning*, also known as *statistical learning*, focuses on
"learning from the data" in order to either predict the outcomes for new
data points (supervised learning) or to find patterns in the data
(unsupervised learning). "Predicting" in this case means obtaining the
values $\\hat{Y}$ for new points which are not in the dataset on which
the model was fitted (the "training data"), using the model parameters
estimated from that dataset. For supervised learning tasks, any of the
regression approaches described above can be considered, noting that in
machine learning, the main goal is having a low prediction error for new
points, as opposed to performing statistical inference. If the number of
predictors is very large, potentially larger than the number of samples,
[regularization
approaches](https://en.wikipedia.org/wiki/Regularization_(mathematics))
like ridge regression, LASSO, or elastic net can be used to reduce
overfitting. Many more approaches exist, some which build on or are
similar to regression. For example, [support vector machines
(SVMs)](https://en.wikipedia.org/wiki/Support_vector_machine#Regression)
for continuous outcomes are equivalent to regression models with a
regularization term and the hinge loss instead of the quadratic loss
function. Similarly, [neural
networks](https://github.com/greenelab/deep-review/blob/master/content/02.intro.md)
can be seen as extensions of linear or logistic regression. It is also
increasingly common in machine learning to use [ensemble
methods](https://en.wikipedia.org/wiki/Ensemble_learning) that combine
several approaches and perform a type of averaging or "voting" to obtain
the final predicted values.