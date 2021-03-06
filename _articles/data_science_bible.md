---
layout: article
title: Plain English Guide to Data Science
date: 2017-03-01
---

# Introduction {#introduction}
This is my attempt to explain some hard-to-grok data science principals in plain english.  It exists because I keep having to look up wikipedia pages that are unnecessarily complicated, and I want a clean and simple explanation for when I inevitably forget everything.

<!--
# Distributions
## Gaussian Distribution
Different sources will tell you different things about the relationship between Gaussian and Normal distributions.  Some say that a Normal distribution is a Gaussian distribution with a mean of 0 and a variance of 1.  Others will call that the *standard normal* distribution, and say Normal and Gaussian are just synonyms.  As far as I can tell, the second camp is technically correct, but unless you are doing something incredibly specific and technical, you can just treat Gaussian and Normal as synonyms.

-->
<!---
## Poisson

## Chi-Squared

## Binomial

## Gamma
-->

# Cost Function {#costfunction}
The [cost function](https://en.wikipedia.org/wiki/Loss_function#Expected_loss), or loss function, is the function that we are trying to minimize when we run a regression.  In other words, it defines what we mean when we say the model is a "good fit," and is closely related to the [deviance](http://stats.stackexchange.com/questions/6581/what-is-deviance-specifically-in-cart-rpart)

A typical cost function would be the [mean squared error](https://en.wikipedia.org/wiki/Mean_squared_error), which is advantageous both because it is symetric (positive and negative errors look the same) and because it penalizes us for having outliers.  A more technical justification for least squares as a cost function is that minimizing least squares is equivalent to maximizing likelihood, assuming normaly distributed errors.  See [here](http://cs229.stanford.edu/notes/cs229-notes1.pdf) for a proof of this, and see the [likelihood](#likelihood) section for a discussion of what the heck that means.

It's worth noting that the cost function is arbitrary- you can minimize whatever you want. It just turns out that certain cost functions have much nicer properties than others.  For example, if you set your cost function to something linear instead of quadratic, you will get bad behavior out of more complicated regression models, because it creates a very flat local gradient curve, making it take much longer (and in some cases with asymtotic activation functions, impossible) to reach global optimas. More on this in the neural network section.

# Linear Regression {#linearregression}
A linear regression is a method to use data you already have to predict outcomes for data you haven't seen yet.  In a linear regression, you have a set of measurements (for example, "height", "weight", "age", etc.) that you use to predict a single numerical outcome (for example, "marathon result time").  You use past observations to come up with a set of coefficients that tell you what formula to use with the input data to predict the outcome.  So for example, you might find that the expected marathon completion time in minutes is equal to 

$${200 - 1.0 * height + 0.5 * weigt + 3.0 * age}$$

In this case, the coefficients are 1.0 for height, 0.5 for weight, and 3.0 for age.  A linear regression finds the set of coefficients that minimizes the sum of squared errors for all observed data.  That is, pick a set of coefficients, and for each observation you have, multiply the coefficients by the data values to come up with a prediction.  Then subtract that prediction from the actual observed outcome to get your "error" (or "residual"). Do that for all your data points, square all the errors you get, and add tem all up to find your sum of squared error.  The set of coefficients that minimizes that sum of squared error is the set of coefficients you get out of a linear regression.

We'll talk more about how you actually go about finding these coefficients in the [regularization](#regularization) section below, but for now just know that we have ways of doing it.

Note that a linear regression assumes that your output is a number (as opposed to a class).  For example, you can't use a linear regression on a person's height and weight to predict whether that person is male or female.  For that, we need other techniques.

<!---
## Single Variable

## Multivariable
-->

## Generalized Linear Model {#generalizedlinearmodel}
I highly recommend the [Wikipedia aritcle](https://en.wikipedia.org/wiki/Generalized_linear_model#Intuition) for GLM, as it is unusually accessible and straightforward.

As the name suggests, the Generalized Linear Model is a generalization of a linear regression model.  GLM assumes some distribution for errors (see [likelihood](#likelihood) section), some linear function that maps inputs to a single numerical value (e.g. linear coefficients x input values), and a "link function."

The link function is the *inverse* of the function that maps our **linear function output** to our **actual prediction for the response variable**.  Put another way, the link function is the function that maps response values to the number predicted by our linear model.  Put yet another way, ["The link function provides the relationship between the linear predictor and the mean of the distribution function"](https://en.wikipedia.org/wiki/Generalized_linear_model#Link_function).  

For example: let's say we have an input vector X with coefficients B.  The output of our linear function would be ${XB} = {x_0 b_0 + x_1 b_1 + x_2 b_2 + ...}$  This result is a single number.  The inverse link function would be some function that maps that single number to some other single number.  In the simplest case, you could just set the link function to ${f(x) = x}$, the identity function.  The inverse of this function is the same as the function itself, so ${ {f}^{-1}(x) = f(x) = x}$.  In this special case, GLM become indistinguishable from a standard linear model; you have some linear function of coefficients and inputs ${XB}$ which maps to a value, and that value is your predictor for y.  We'll talk about other link functions in the below sections.

The nice thing about GLM, and the reason you want a link function other than the identity function, is that it lets us achieve non-linear behavior in our model while still having the simplicity of only needing to estimate linear paramaters

In order to fit the coefficients of a GLM, we find the model that minimizes the cost function for a given set of data.  We typically use [numerical optimization](#numericaloptimization) techiques for this.

## Logistic Regression {#logisticregression}

A logistic regression is a regression primarily used to predict a binary class - e.g. predicting "male" or "female" from some set of inputs. It turns out a logistic regression is just a GLM with a link function of ${ {XB} = {ln({ {u}\over{1 - u}})}}$ (called the "logit" function or the "log-odds" function), which gives an inverse link function of ${ {u} = { {1}\over{1+e^{-XB}}}}$.  When you need to make a prediction for y, you find the linear combination of B's and X's, and then you perform the inverse link function on your result to get your prediction.

For example, if you have a GLM model with two inputs (${x_1}$ and ${x_2}$) and three coefficients (${b_0}$, ${b_1}$, and ${b_2}$, where ${b_0} is the bias), then in order to make a prediction for y from a new set of inputs, you perform the following procedure:

1. Use the coefficients in your model to predict n 
  $${n = b_0 + x_1 b_1 + x_2 b_2}$$
2. Transform your value n using the inverse link function
  $${y} = { {1}\over{1+{e}^{-n}}}$$
3. The result of that transformation is your prediction for y

In the above case, our linear mapping function is just the linear combination ${n} = {b_0 + x_1 b_1 + x_2 b_2}$.  The inverse link function is  ${ {1}\over{1+{e}^{-n}}}$.

This has some nice properties.  It means that your response will always range from 0 to 1, which is nice because that's what probabilities do.  It also means our b's have a straightforward interpretation: ${e}^{b_i}$ is the mulitpicative change in **odds** (not probability) associated with a one-unit increase in ${x_i}$.

For example: if ${b_i}$ is 1.5, then a 1 unit change in ${x_i}$ corresponds with a ${e}^{1.5}={4.48}$ times increase in the odds of whatever you're predicting.  If your default odds are ${1}\over{10}$, then a 1 unit increase in ${x_i}$ means your new odds are ${4.48}\over{10}$, and your probability goes from ${1}\over{1 + 10}$ to ${4.48}\over{4.48 + 10}$. See [here](http://www.ats.ucla.edu/stat/mult_pkg/faq/general/odds_ratio.htm) for a great explanation of the math.

It's important to note here that the derivative of the inverse link function (lets call it ${g}$, so ${g} = {f}^{-1}(X)$) is given as ${g'} = {g(1-g) dg}$.  This is useful because some of the coefficient estimation techniques [discussed below](#gradientdescent) require us to know the slope of the the response with respect to the input variables.  Since it's really easy to calculate the derivative of the inverse logit function, it makes it particularly useful for these kinds of applications. 

My big takeaway, and the thing that isn't really discussed very well in most literature, is that **the logit function is totally arbitrary**.  It happens to have some nice properties- bounded between 0 and 1, easy to differentiate, etc. - but there's no a priori reason to think this is the optimal function for every data distribution.  It's really just a regular old GLM- it's mapping X's to numeric values, then transforming those numeric values to be between 1 and 0.  That's it. No magic at all.  No strong theoretical basis. Just a convenient function.

The biggest implication of this (to me, anyway), is that when you get a prediction from a logistic regression, that's our best estimate of the probability from a *linear model we built to estimate the log of the odds*.  If the log odds don't actually go linearly, then our model is going to be bad.  Full stop.  I've built models on data before that predict a probability of 10% given a common set of inputs, but then I look at the actual incidence for those inputs and it's more like 3%, and I'm confused- shouldn't my model be relatively close to the mean at every given point?  Well, no, there's no reason to think it would bue unless you assume log-odds is linear.  If the relationship between X and log odds isn't linear, then a logit transform isn't going to do much good- or it least is going to give you systemically biased predictions.

You can read a whole lot more on logistic regression [here](http://www.win-vector.com/blog/2011/09/the-simpler-derivation-of-logistic-regression/) and [here](http://www.stat.cmu.edu/~cshalizi/350/lectures/26/lecture-26.pdf).

<!---
## Multinomial Logistic

P(Y = c | X) = exp(B_0c + B_cX)/SUM(exp(B_0i + B_iX))

## Ordinal Regression

## Probit Regression
-->

# Model Fit {#modelfit}

## Likelihood {#likelihood}
The likelihood is the probabitlity of seeing exactly what we've seen in our data *given that a particular model is true*.

Suppose we have a model that classifies inputs into bucket "male" or bucket "female" based on their height and weight (see Classification Models section).  The response is "percentage chance of this person being male." Now suppose we have the following predictions:

```{r kable, echo=FALSE}
male_female_input=data.frame(prediction_male=c(0.7, 0.85, 0.1), height=c(70, 71, 65), weight=c(160, 185, 120))

panderOptions("digits", 2)
pander(male_female_input)
```

Now suppose we actually know the real answer about the gender of these observations:

```{r, echo=FALSE}
male_female_input = data.frame(male_female_input, gender=c("female", "male", "male"))

panderOptions("digits", 2)
pander(male_female_input)
```
Now we can ask ourselves the question: how likely were we to see this exact set of results, *given that our model is true*?  It's easy to calculate- the probability of the first being female is 1 - p_1(male), the probably of the second being male is p_2(male), and the probability of the third being male is p_3(male).  In this case, that means:

```{r}
(1-0.7)*(0.85)*(0.1)
```

Hence, if our underlying probability model is correct, we will see this exact distribution 2.55% of the time.  That is our 'likelihood'.

This is a lot more intuitive (at least for me) when dealing with classification models that give you percentages, rather than estimation models that give you expected values.  But the actual logic holds for linear regressions too- since linear regressions assume some underlying distribution, we just need to figure out the [probability density function](https://en.wikipedia.org/wiki/Probability_density_function) for that distribution, and we can compute the likelihood of the exact observations we made.  See [this summary](http://www.statlect.com/normal_distribution_maximum_likelihood.htm) if you still need convincing.  

Notice that we have to know the error distribution in order to say anything about likelihood.  If we change our assumptions about the error distribution, we have to change what we say about the likelihood of the observations.

Also notice that adding more observations to our training set *always makes our likelihood go down*.  This makes perfect sense- unless the probability of the next observation is exactly 1 or exactly 0, then it's always less likely that we'll see this current distribution *plus one extra observation* than it is that we'll see this current distribution.

When talking about [maximum-likelihood](https://en.wikipedia.org/wiki/Maximum_likelihood#Properties), you may sometimes see a formula written (with different variable names) as $${l(M \lvert x)} = { {1}\over{n}} \sum{ln(f(x \lvert M))}$$

This is kind of confusing, right?  We just talked about how the likelihood of a model is the likelihood of each of the events happening multiplied together.  However, notice the [product rule](https://en.wikipedia.org/wiki/Logarithm#Product.2C_quotient.2C_power_and_root) of logarithms: log(a) + log(b) = log(a * b).  So we could rewrite this formulation as:

$${l(M \lvert x)} = { {1}\over{n}} {ln(f(x_1 \lvert M)  * f(x_2 \lvert M) * f(x_3 \lvert M)...)}$$

In this case, l is actually the average [log-likelihood](https://en.wikipedia.org/wiki/Likelihood_function#Log-likelihood) function where f is the simple likelihood function.  We tend to work with log likelihood instead of likelihood mostly because it has some useful properties when it comes time to start taking derivatives. Since it is monotonically increasing with likelihood, it preserves ordering, and maximizing likelihood is equivalent to maximizing log-likelihood.

In general, when people talk about likelihood they are typically talking about log-likelihood.  It's a distinction kind of like Gaussian vs. Normal- some people care a lot, but in general getting too wrapped up in likelihood doesn't matter, since maximizing likelihood is equivalent to maximizing log-likelihood.

Now here's the three important facts about likelihood:
1. Maximizing likelihood is the same as minimizing deviance
2. Minimizing sum of least squares in the same as maximizing likelihood for a linear regression (assuming a normal distribution of errors)
3. The way we fit General Linear Models (see [General Linear Model](#generallinearmodel) section) is by maximizing the likelihood.

## Deviance {#deviance}
Sources on deviance are quite confusing.  Simplye put, deviance is given as:

$$-2 * {log(Likelihood(M0)) - log(likelihood(M1))}$$

But you will sometimes instead see it given as:

$$-2{log(Likelihood(M)}$$

Technically speaking, deviance is a fact about the *difference* between two models, so you can't calculate the deviance for a single model all by itself.  However, consider a "saturated" model, in which the number of observations is the same as the number of parameters.  In this model, we can perfectly predict every observation- there is no error.  Hence, the likelihood(M) is just 1!  Well log(1) is just 0, so the first equation simplifies to the second.

You may of course use a different model for M1 than the saturated model.  In particular, it is common to use M1 with a whole bunch of parameters, and M0 with some subset of those parameters, and then look at the deviance between the two.

# Regularization {#regularization}

<!---
## lasso regularization

## Cross Validation

## Optimization Functions
-->
#Numerical Optimization {#numericaloptimization}


##Closed Form Solution {#closedformsolution}
In the case of the standard OLS linear regression, there is a closed form solution based on matrix math.  The derivation is interesting, but here's the kicker:

$$θ = (X^{T} X)^{−1} X^{T} y$$

In practice, we don't compute it this way because it's computationally inefficient.  Or so the literature says.

Moreover, this method isn't useful in a lot of other contexts- for example, any link function that doesn't have a closed form solution, or any algorithm that employs regularization in model construction.  For those, we need numerical optimization techniques

## Newton's Method {#newtonsmethod}
You can look up the details how how it works, but the general gist is that finding the global minimum of a function of a single variable f(n) can be found by iteratively updating w as follows:

$$ w_n+1 = w_n - {f'(w_n)}\over{f''(w_n)}$$

Likewise, a function of multiple variables can be minimized by iteratively updating the vector W as follows:

$$W_n+1 = W_n - H^{-1}(W_n) * delta(f(W_n))$$

Where delta(f(W_n)) is the gradient of f (a vector of the first partial derivative) and H is the Hessian matrix (the n x n matrix of partial second derivatives)

## Gradient Descent {#gradientdescent}
see [here](http://cs229.stanford.edu/notes/cs229-notes1.pdf)

## Stochastic Gradient Descent {#stochasticgradientdescent}
1. Pick starting values for all coefficients
2. Pick an observation from our training data
3. Find the partial derivative of the likelihood function with respect to each coefficient
4. Calculate the "step size" by which we're going to change the coefficients (this could be a constant, or a value depending on the number of observations we've seen so far, or a per-coefficient value depending on the magnitude of the partial derivative and the number of positive observations, etc.  Any function will do, and most are tested experimentally)
5. Change each coefficient along its partial derivative by the calculated step size (getting us slightly closer to the maximum)
6. If some condition is met, quit.  Otherwise, go to step 2

# Nonlinear Models {#nonlinearmodels}
Nonlinear models come in two flavors, just like linear models-classification and regression.  We discuss both types below

<!---
## K nearest neighbors

## Trees

## Forests

## Boosting

## Classification Trees

## K-Means
-->

## Principal Component Analysis {#principalcomponentanalysis}
PCA is a concept that doesn't fit super nicely into any category.  It's really an exploratory method, helping us descover facts oabout our data that are not obvious from the way we collect it.  Put another way, sometimes the particular form of data that we have is an artifact of our particular method of collection.  If we had collected some subset of our data in a slightly different way (with a different orthogonal basis), then we would get a much simpler model.  Google Research has an excellent intuitive summary of PCA [here](http://arxiv.org/pdf/1404.1100.pdf)

One interesting point worth mentioning is that PCA assumes linearity.  As it says in the above linked paper, "with this assumption PCA is now limited to re-expressing the data as a linear combination of its basis vectors." 

Conceptually, PCA uses the covarince matrix and Singular Value Decomposition to figure out underlying orthogonal variables that explain the most amount of variance in the responses in our data.  So if X_1, X_2, and X_3 are all correlated, it could be that by performing some transform and re-basing our data, we capture all of this correlation into a single variable which explains everything that X_1, X_2, and X_3 used to explain.  In this case we have accomplished some dimensional reduction, and this changes the dynamics of any sort of regularized regression we may want to run.

Another way to think about PCA is that covariance is an expression of *redundancy in our data*.  By recasting our data using a basis where none of our inputs covary, we are getting rid of that redundancy- simplifying and shrinking the number of inputs to be the number of *real underlying inputs* that actually matter.

# Neural Networks {#neuralnetworks}

# Kurtosis {#kurtosis}
[Kurtosis](https://en.wikipedia.org/wiki/Kurtosis) is a measure of the "sharpness" of a distribution. It can also be thought of as a measure of "narrowness" of a distribution.  Sharpening a distribution has the effect of fattening the tails (that extra area under the curve has to go somewhere!) 

A Normal distribution is simply a _____ distribution with a kurtosis of 3.In practice, many real-world distributions appear to be normal, but in fact have a different kurtosis (financial returns on the stock market being a famous example- we see more long-tail events than we would expect in market data)

<!--
## Recurrent

## Convolutional

## Deep

# Model Composition
Tree + logistic
xgboost
feature hacking
bagging and binning

#Further Reading

Facebook paper
Parametric vs non-parametric
  PCA is non-parametric
  Additive models are non-parametric
Singular Value Decomposition
-->