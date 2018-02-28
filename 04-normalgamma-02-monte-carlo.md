## Monte Carlo Inference  {#sec:NG-MC}







In this section, we will illustrate Monte Carlo sampling from posterior distributions and how it can be used for inference. In Section \@ref(sec:normal-gamma), we can get the posterior distributions for the precision (inverse variance), and the mean given the precision. Then the marginal distribution of the mean can be obtained via integration.

Here is a recap of the joint posterior distribution for the mean $\mu$ and the precision $\phi = 1/\sigma^2$:

* Conditional posterior $\mu \mid \data, \sigma^2  \sim  \No(m_n, \sigma^2/n_n)$
* Marginal posterior $1/\sigma^2 = \phi \mid \data   \sim   \Ga(v_n/2,s^2_n v_n/2)$
* Marginal posterior $\mu \mid \data \sim \St(v_n, m_n, s^2_n/n_n)$

What if we are interested in the distribution of the standard deviation $\sigma$ itself, or other transformations of the parameters? There may not be a closed-form expression for the distributions.

However, it turns out that **Monte Carlo sampling** is an easy way to make an inference, when we cannot analytically calculate distributions of parameters, expectations, or probabilities. Monte Carlo methods are computational algorithms that rely on repeated random sampling to calculate numerical results. The name refers to the famous Monte Carlo Casino in Monaco, home to games of chance such as Roulette.

Let's start with a case where we know the posterior distribution.


For posterior inference about the precision $\phi$ using Monte Carlo simulation, we generate $S$ random samples from the posterior distribution:

$$\phi^{(1)},\phi^{(2)},\cdots,\phi^{(S)} \iid \Ga(v_n/2,s^2_n v_n/2)$$

The term **iid** stands for **i**ndependent and **i**dentically **d**istributed. In other words, the $S$ draws of $\phi$ are independent and identically distributed from the gamma distribution.

Then the empirical distribution of the $S$ samples is used to approximate the actual posterior distribution. From the samples, the sample mean of the draws of $\phi$ can be used to approximate the posterior mean of $\phi$.

Likewise, we can calculate probabilities, quantiles and other functions using the samples from the posterior distribution. For example, if we want to calculate the posterior expectation of some function of $\phi$, written as $g(\phi)$, we can approximate that by taking the average of the function, and evaluate it at the $S$ draws of $\phi$, written as $\frac{1}{S}\sum^S_{i=1}g(\phi^{(i)})$.

The approximation improves as the size of the Monte Carlo simulation $S$ increases.

$$\frac{1}{S}\sum^S_{i=1}g(\phi^{(i)}) \rightarrow E(g(\phi \mid \data))$$

**Example**

We will apply this to the tap water example. To start, we will set a random seed, which allows the results to be replicated. To generate 1,000 draws from the gamma posterior distribution, we use the `rgamma` function `R` with the posterior hyperparameters from last time.


```r
data(tapwater)
m_0 = 35; 
n_0 = 25; 
s2_0 = 156.25;
v_0 = n_0 - 1
Y = tapwater$tthm
ybar = mean(Y)
s2 = round(var(Y),1)
n = length(Y)
n_n = n_0 + n
m_n = round((n*ybar + n_0*m_0)/n_n, 1)
v_n = v_0 + n
s2_n = round( ((n-1)*s2 + v_0*s2_0 + n_0*n*(m_0 - ybar)^2/n_n)/v_n, 1)
L = qt(.025, v_n)*sqrt(s2_n/n_n) + m_n
U = qt(.975, v_n)*sqrt(s2_n/n_n) + m_n
```


```r
set.seed(8675309)
phi = rgamma(1000, shape = v_n/2, rate=s2_n*v_n/2)
```

Figure \@ref(fig:phi-plot) shows the histogram of the 1,000 draws of $\phi$ generated from the Monte Carlo simulation, representing the empirical distribution. The orange line represents the actual gamma posterior density.

\begin{figure}

{\centering \includegraphics{04-normalgamma-02-monte-carlo_files/figure-latex/phi-plot-1} 

}

\caption{Empirical distribution of the tap water example}(\#fig:phi-plot)
\end{figure}

Try changing the random seed or increasing the number of simulations, and see how the approximation changes.

We will now use Monte Carlo simulations to approximate the distribution of $\sigma$. Since $\sigma = 1/\sqrt{\phi}$, we can apply the transformation to the 1,000 draws of $\phi$ to obtain a random sample of $\sigma$. We can then estimate the posterior mean of $\sigma$ by calculating the sample mean of the 1,000 draws.


```r
sigma = 1/sqrt(phi)
mean(sigma) # posterior mean of sigma
```

```
## [1] 21.814
```

Similarly, we can obtain a 95% credible interval for $\sigma$ by finding the sample quantiles of the distribution.


```r
quantile(sigma, c(0.025, 0.975))
```

```
##     2.5%    97.5% 
## 18.20700 26.53736
```

**Summary**

To recap, we have introduced the powerful method of Monte Carlo simulation for posterior inference. Monte Carlo methods provide estimates of expectations, probabilities, and quantiles of distributions from the simulated values. Monte Carlo simulation also allows us to approximate distributions of functions of the parameters, or the transformations of the parameters.

Next we will discuss predictive distributions and show how Monte Carlo simulation may be used to help choose prior hyperparameters, using the prior predictive distribution of data.