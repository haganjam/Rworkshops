Linear model assumptions
================
James G. Hagan
2024-01-11

### Checking the assumptions of a linear model

Over the last four years, I’ve taught on bio-statistics courses which
focus on general linear models. And, I’ve come to discover that one of
the most difficult things that I’ve tried to teach is how to check the
assumptions of linear models. The reason for this (I think) is that
there is no clear, standard recipe for doing this. Moreover, we tend to
tell students to do things like: “check if the residuals are normally
distributed”. However, at the start it’s not that easy to decide when
something is normal enough.

This feeling of uncertainty surrounding when the residuals are “normal
enough” or when it seems that some pattern in the residuals is “too
much” leads many to rely on statistical tests for checking these
assumptions. For example, it is reasonably common for students to ask me
whether we can use a Shapiro-Wilk’s test for normality to test if the
residuals are normally distributed. This, in and of itself, is not a
terrible idea but the problem with these tests for verifying assumptions
is statistical power. If I use a Shapiro-Wilk’s test on some residuals
and I fail to reject the null hypothesis of normality, does that mean
the data are normal or does it simply mean that I didn’t have enough
statistical power to reject the null hypothesis? This becomes especially
problematic with small sample sizes where the test will inherently have
low statistical power.

One thing that I found helped students was to show them clear examples
of what non-normal residuals look like by simulating data, for example,
that is Poisson distributed or Log-Normally distributed. So, in this
tutorial, I will go through the assumptions of linear models, why we
have to test the assumptions and then use some simulations to show what
residuals plots look like when the assumptions of regression are
violated.

### What are the assumptions of linear models?

I have been rather underwhelmed by many of the short blog posts and
articles on this topic. They seem to treat these assumptions in a very
superficial way without really going into the details about why we need
to test different assumptions. Here, I won’t go into all the details.
The details are provided in so many textbooks. I can recommend Quinn and
Keough (2002) which, in my view, gives solid amount of detail and is
presented very clearly. My goal here is to provide some intuition of how
a linear model is set-up and why we have to make certain assumptions and
how important this is for us to make inference.

Let’s imagine a population of 10 000 Cape fur seals (*Arctocephalus
pusillus*) each of which has some body weight ($y_i$). Let’s then say
that there is a relationship between a seal’s body weight ($y_i$) and
the number of fish ($x_i$) a given seal eats. We can write this
population-level model as:

$$
y_i = \beta_0 + \beta_1x_i + \epsilon_i
$$

In this model, the $\beta_0$ parameter is the population intercept
which, in this case, can be interpreted as the mean value of the
distribution of Y when $x_i$ is equal to zero. Similarly, $\beta_1$ is
the population slope which describes the change in Y for very unit
change in X. Finally, $\epsilon_i$ is the random error of the $i^{th}$
observation which effectively measures the difference between each
observed $y_i$ and the mean or expected value of $y_i$ which is what is
predicted by the population regression line.

Let’s simulate this scenario. To do this, for 10 000 seals (N), we draw
a number of fish eaten ($x_i$) from a Poisson distribution:

``` r
# set the seed for reproducibility
set.seed(454)

# set the number of seals in the population
N <- 10000

# draw 10 000 xi values from a Poisson distribution with lambda = 10
xi <- rpois(n = N, lambda = 10)
```

Cape fur seals generally weight between 100 and 300 kg. So, if we
imagine that an individual seal that weighs 100 kg is underfed, then we
can set the $\beta_0$ population-level intercept to 100 because this
represents the average weight of a seal when the number of fish it has
eaten is zero.

We can then imagine that, for every fish a seal eats, it gains 5 kg on
average. Therefore, we set the $\beta_1$ population-level slope to 5.
Finally, there is some random error ($\epsilon_i$) around the mean Y
(seal body weight) for every X value (number of fish eaten). We assume
that this random error ($\epsilon_i$) is normally distributed around
zero.

Using these parameters, let’s simulate the $y_i$ values in the
population.

``` r
# set the population-level beta parameters
B0 <- 100
B1 <- 5

# simulate the yi values for the 10 000 seals with some normally distributed error
yi <- B0 + (B1*xi) + rnorm(n = N, 0, 5)
```

Now, we can plot the relationship between the number of fish eaten by
each seal and the seal’s body weight in the population:

``` r
plot(yi ~ xi, ylab = "Seal weight (kg)", xlab = "Number of fish eaten")
```

![](01-linear-model-assumptions_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

So let’s now imagine that we wanted to *estimate* the relationship
between the number of fish a seal eats and its weight. For this, we need
to take a random sample of seals from this population and then fit a
model to this sample. For this example, we’ll take a sample of 100
seals:

``` r
# take a random sample of seals from the population
n_id <- sample(1:N, size = 100)

# get the relevant xi values
xi_s <- xi[n_id]

# get the relevant yi values
yi_s <- yi[n_id]

# plot the relationship between xi_s and yi_s of the sample
plot(xi_s, yi_s, ylab = "Seal weight (kg)", xlab = "Number of fish eaten")
```

![](01-linear-model-assumptions_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

We can now fit a linear regression model to these sample data and try to
estimate the population-level parameters that we simulated ($\beta_0$
and $\beta_1$). A linear regression model takes the following form:

$$
y_i = b_0 + b_1x_i + e_i
$$

Here, the $b_0$ and $b_1$ are estimates of the population-level
parameters ($\beta_0$ and $\beta_1$) and the $e_i$ values are the model
residuals i.e. the difference between the observed $y_i$ and the
predicted value which is $\hat{y_i}$

Let’s fit this model using R’s built-in *lm()* function which uses
ordinary least squares:

``` r
# fit the linear regression model
lm1 <- lm(yi_s ~ xi_s)

# check the estimated model coefficients
print(lm1)
```

    ## 
    ## Call:
    ## lm(formula = yi_s ~ xi_s)
    ## 
    ## Coefficients:
    ## (Intercept)         xi_s  
    ##     102.516        4.635

As we can see from the above output, the $b_0$ (i.e. Intercept) and
$b_1$ (i.e. xi_s) parameter estimates outputted by the model are very
close to the population-level $\beta_0$ and $\beta_1$ parameters that we
simulated.

If we look a bit further at the output of this model, we get two more
pieces of information that are probably relatively familiar to most
people: P-values and confidence intervals.

First, when we use the *summary()* function on an *lm()* object, we get
statistical tests of the null hypothesis that the estimated parameters
$b_0$ and $b_1$ are zero. If the P-value is less than 0.05 (purely from
statistical convention), then we reject the null hypothesis and we say
that parameters are different from zero. Let’s look at this output for
this model:

``` r
# check the summary output
summary(lm1)
```

    ## 
    ## Call:
    ## lm(formula = yi_s ~ xi_s)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -14.5809  -3.2405  -0.5571   3.2959  12.6680 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 102.5165     1.7630   58.15   <2e-16 ***
    ## xi_s          4.6352     0.1734   26.73   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 5.389 on 98 degrees of freedom
    ## Multiple R-squared:  0.8794, Adjusted R-squared:  0.8782 
    ## F-statistic: 714.5 on 1 and 98 DF,  p-value: < 2.2e-16

As we can see from this output, the P-values associated with both $b_0$
and $b_1$ are less than 0.05 and so we conclude that the null hypothesis
is rejected: these parameters differ from zero.

We can also check the confidence intervals around these parameters. We
usually calculate a 95% confidence interval but, like the 0.05
significance-level of a P-value, is purely conventional. Let’s look at
the confidence intervals for $b_0$ and $b_1$:

``` r
# check the confidence intervals around b0 and b1
confint(lm1)
```

    ##                2.5 %     97.5 %
    ## (Intercept) 99.01790 106.015088
    ## xi_s         4.29109   4.979341

So, as we can see, the 95% confidence interval is a range representing
the uncertainty in our parameter estimates for $b_0$ and $b_1$. How to
properly interpret a confidence interval is not very straightforward.
I’m sure you’ve heard some phrase like: “there is a 95% probability that
the true parameter value is within this confidence interval”. This is
not strictly true. The proper interpretation of a confidence interval is
as follows. Imagine we could redo our study 1000 times i.e. and take 100
seals from the population 1000 separate times and each time estimate the
95% confidence interval for the two parameters. What the confidence
interval says is that, in 950 out of 1000 (i.e. 95%) of those studies,
the calculated confidence intervals would contain the true parameter
values.

You’re probably thinking: “what does this have to do with checking
linear model assumptions?”. A fair point. Admittedly, this has been a
pretty long build-up. However, it is important because for the P-values
and confidence intervals we just looked at to be valid (i.e. to do what
they are supposed to do), we need to make some assumptions about the
error term in the population-level regression ($\epsilon$). When we test
assumptions, we are effectively trying to verify certain features of
this error term ($\epsilon$) which then allows us to make valid
inferences based on P-values and confidence intervals. If this is
confusing, don’t worry! We are going to simulate all of this and it will
hopefully become clearer.

### What are the assumptions of linear regression and why do we need them?

Let’s write out the population-level regression along with the
assumptions that we need to make about the error term ($\epsilon$).

$$
y_i = \beta_0 + \beta_1x_i + \epsilon_i \\
\epsilon_i \sim Normal(0, \sigma^2)
$$

So what this says is that we have to make the assumption that the error
term ($\epsilon_i$) is normally distributed around 0 which is the first
assumption. We also have to assume that the variance ($\sigma^2$) is
constant and therefore does not change depending on the value of $x_i$.
We can see this from the above equation because there is only one
($\sigma^2$). There is a third assumption which is that the individual
errors ($\epsilon_i$) are independent.

When these assumptions are met, our P-values and confidence intervals
are valid. Let’s use simulations to prove this to ourselves. We’re going
to focus on the confidence intervals because it is easier to illustrate.
A valid confidence interval means that if we could perform a study 1000
times, the 95% confidence interval of an estimated parameter would
contain the true population-level parameter in 950 of those 1000 studies
(i.e. 95%)

In our simulated example, I specified that the error term ($\epsilon_i$)
was normally distributed around 0 and with a constant variance
($\sigma^2$) of 25 (go look back at the code) irrespective of the $x_i$
value. Thus, in this simulated example, the assumptions are met and so
the confidence intervals should be valid. Let’s see if this is the case.

In this block of code, all we are going to do is take a random sample of
seals from the population of 10 000 seals, fit the linear model,
calculate the confidence intervals for the $b_0$ and $b_1$ parameters
and check whether the population-level parameters $\beta_0$ and
$\beta_1$ are within those confidence intervals. We will then do this
1000 times.

``` r
# set-up the number of studies
N_studies <- 1000

# set-up output vectors
b0_vec <- vector(length = N_studies)
b1_vec <- vector(length = N_studies)

for(i in 1:N_studies) {
  
  # take a random sample of seals from the population
n_id <- sample(1:N, size = 100)

# get the relevant xi values
xi_s <- xi[n_id]

# get the relevant yi values
yi_s <- yi[n_id]

# fit the linear model
lmx <- lm(yi_s ~ xi_s)

# calculate the confidence intervals
cix <- confint(lmx)

# test if B0 is within the confidence interval
b0_vec[i] <- (cix[1,][1] < B0) && (B0 < cix[1,][2])

# test if B1 is within the confidence interval
b1_vec[i] <- (cix[2,][1] < B1) && (B1 < cix[2,][2])
  
}
```

Now, using this simulation, we can check how many times out of the 1000
times we did this study that the the confidence intervals that we
calculated contained the populations parameters:

``` r
# how many of the calculated confidence intervals contained the true population parameter B0:
sum(b0_vec)
```

    ## [1] 951

``` r
# how many of the calculated confidence intervals contained the true population parameter B1:
sum(b1_vec)
```

    ## [1] 949

Whenever I do these simulations, I always find it quite amazing to see
how close to 95% we get. In this simulation, the calculated confidence
intervals for the $b_0$ parameter contained the true population-level
$\beta_0$ parameter in 951 out of the 1000 simulated studies
(i.e. almost exactly 95%). For the $b_1$ parameter, it was 949.

Now let’s look at a case where the assumptions are violated and see what
happens. To do this, we will simulate the same population of 10 000
seals but rather than the error term being normally distributed around 0
with a variance of 25, we will do the following. The error term will be
distributed normally with a mean of 0.1 and the variance will increase
with $x_i$ by a factor of 0.75.

``` r
# set the number of seals in the population
N <- 10000

# draw 10 000 xi values from a Poisson distribution with lambda = 10
xi <- rpois(n = N, lambda = 10)

# set the population-level beta parameters
B0 <- 100
B1 <- 5

# simulate the yi values for the 10 000 seals with some lognormal error
yi <- B0 + (B1*xi) + rnorm(n = N, mean = 0.1, sd = 0.75*xi)
```

The consequence of this which is easily seen when plotting $y_i$ and
$x_i$ is that the variation of $y_i$ for a given $x_i$ increases as
$x_i$ increases. Moreover, the error term is not centered on zero
although this is difficult to see in the plot.

``` r
# plot the results
plot(yi ~ xi)
```

![](01-linear-model-assumptions_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Given that we know we have violated two out of the three assumptions
about the error term ($\epsilon_i$), we would not expect our confidence
intervals to perform as well as in the previous simulation. Let’s see if
this is indeed the case. To do this, we do the same thing as previously:
take a random sample of 100 seals from the population of 10 000 seals,
fit the linear model, calculate the confidence intervals for the $b_0$
and $b_1$ parameters and check whether the population-level parameters
$\beta_0$ and $\beta_1$ are within those confidence intervals. Do this
1000 times.

``` r
# set-up the number of studies
N_studies <- 1000

# set-up output vectors
b0_vec <- vector(length = N_studies)
b1_vec <- vector(length = N_studies)

for(i in 1:N_studies) {
  
  # take a random sample of seals from the population
n_id <- sample(1:N, size = 100)

# get the relevant xi values
xi_s <- xi[n_id]

# get the relevant yi values
yi_s <- yi[n_id]

# fit the linear model
lmx <- lm(yi_s ~ xi_s)

# calculate the confidence intervals
cix <- confint(lmx)

# test if B0 is within the confidence interval
b0_vec[i] <- (cix[1,][1] < B0) && (B0 < cix[1,][2])

# test if B1 is within the confidence interval
b1_vec[i] <- (cix[2,][1] < B1) && (B1 < cix[2,][2])
  
}
```

Check how many times out of the 1000 times we did this study that the
the confidence intervals that we calculated contained the populations
parameters:

``` r
# how many of the calculated confidence intervals contained the true population parameter B0:
sum(b0_vec)
```

    ## [1] 965

``` r
# how many of the calculated confidence intervals contained the true population parameter B1:
sum(b1_vec)
```

    ## [1] 926

Interestingly, despite violating these assumptions, the calculated
confidence intervals for the $b_0$ parameter contained the true
population-level $\beta_0$ parameter in 965 out of the 1000 simulated
studies (i.e. not terribly far from 95%). However, for the $b_1$
parameter, it was only 926 which means that the 95% confidence intervals
we calculated are not valid.

Hopefully, these simulations give you a sense of why we need to check
our assumptions. As is clear from this second example, we cannot
interpret the 95% confidence interval as a 95% confidence interval when
we violate the assumptions.

### How do we check these assumptions?

You may be thinking: “Mate, we generally only have one sample from the
population. How in God’s name are we then supposed know about the
distribution of the error term in the population?”

A fair question. Fortunately, there are some ways for us to try and
verify these assumptions. All of these methods revolve around the
*residuals* of the model. What are the residuals?

If we go back to our model equation from above:

$$
y_i = b_0 + b_1x_i + e_i
$$

As described previously, the $b_0$ and $b_1$ are estimates of the
population-level parameters ($\beta_0$ and $\beta_1$) and the $e_i$
values are the model residuals i.e. the difference between the observed
$y_i$ and the predicted value which is $\hat{y_i}$

You can imagine that the residuals ($e_i$) from our sample might contain
some information about the error term in the population ($\epsilon$).
Therefore, the standard way to check linear model assumptions is to
graphically analyse the residuals to see if they look like they come
from a population where the error term ($\epsilon$) is in fact normally
distributed with constant variance.

How do we do this? Well, one of the first things that we can do is check
if the residuals are normally distributed. Why does this work?

*to be continued*
