---
layout: single
title: "Ch3. Bootstrap"
permalink: /coursework/STATS509/study/bootstrap/
comments: false
author_profile: true
read_time: true
toc: true
toc_label: "Table of Contents"
toc_sticky: false
categories: ["STATS509"]
---

```R
options(warn=0)
library(ggplot2)
```

## [ 3.1. Concept ]

Suppose that $X_1, ..., X_n$ are iid samples from a distribution function $F(x;\theta)$, where $\theta \in \Theta$ is an unknonw parameter. Also suppose that we have as statistic $\hat \theta = S(X_1, ..., X_n) \approx \theta$, which is a good (i.e. consistent) estimate of $\theta$.        
We want a confidence interval for $\theta$, but it is just too complicated to obtain. What to do?

Some intuition:     
- Let's define the empirical CDF: $\hat F_n(x) = \frac{1}{n} \sum_{i = 1}^{n}{\mathbb{1}(X_i \leq x)}$  
- By Glivenko-Cantelli, we have that $\underset{x \in \mathbb{R}}{sup} |\hat F_n(x) - F(x)| \rightarrow 0$, as $n \rightarrow \infty$    
  That is, for large samples, $\hat F_n (x) \approx F(x)$

Let's consider following resampling method: 
- Generate a random sample $X_1^*, ..., X_n^*$ from $\hat F_n$. This will be called a bootstrap sample from $X_1, ..., X_n$. This is equivalent to drawing with replacement from $X_1, ..., X_n$ 
- Compute $\hat \theta^* = S(X_1^*, ..., X_n^*)$
- Repeate the above steps independently to generate B independent bootstrap samples    
      
  $$ 
  X_{b, 1}^{*}, ..., X_{b, n}^{*}, \quad b = 1, ..., B \\
  $$
      
  and compute the corresponding statistcs

$$
\hat \theta_{b}^{*}, \quad b = 1, ..., B
$$

Why is this related to confidence interval for $\theta$?
- Think of $X_1, ..., X_n$ as the population.     
  Then, $\hat \theta$ is for $\theta$ is what $\hat \theta^*$ is for $\hat \theta$.   
  That is, 
      
  $$
  \hat \theta - \theta \overset{d} \approx \hat \theta ^ * - \hat \theta |X_1, ..., X_n \tag{1}\\
  $$

- This means that we can get the distrbution of $\hat \theta ^ * - \theta$ through resampling. 
- For fixed $\alpha \in (0, 1)$ and large enough B, we can compute $q_L$ and $q_U$ as the lower and upper $\alpha \, / \, 2$ empirical quantiles of $\hat \theta ^ * - \hat \theta$, so that 
      
  $$
  P_*(q_L(\alpha \, / \, 2) \leq \hat \theta^* - \hat \theta \leq q_U(\alpha \, / \, 2)) \approx 1 - \alpha \\
  $$

  And by (1) 
      
  $$
  P(q_L(\alpha \, / \, 2) \leq \hat \theta - \theta \leq q_U(\alpha \, / \, 2)) \approx 1 - \alpha \\
   \rightarrow P(\hat \theta - q_U(\alpha \, / \, 2) \leq \theta \leq \hat \theta - q_L(\alpha \, / \, 2)) \approx 1 - \alpha
  $$

$\textbf{Example 1}$: Bootstrap works!


```R
# return alpha% confidence interval of the mean of population distribution
boot_mean_CI = function(data, B, alpha, m = length(data)){
    theta_hat = mean(data)
    n = length(data)
    empiricial_delta = c()
    
    for (i in c(1:B)){ # We will generate B bootstrap samples  
        theta_hat_star = mean(sample(data, size = m, replace = TRUE)) # Generate bootstrap sample from the data and 
                                                                      # calculate theta_hat_star for each bootstrap samples
        empiricial_delta = c(empiricial_delta, sqrt(m / n) * (theta_hat_star - theta_hat))
    }    
    
    q_L = quantile(empiricial_delta, alpha / 2)
    q_U = quantile(empiricial_delta, 1 - (alpha / 2))
    
    res = list("CI_lower" = theta_hat - q_U, 
               "CI_upper" = theta_hat - q_L)

    return(res)
}
```

Above function calculate and return estimated $\alpha$% confidence interval for the mean of the populaion distribution  by the bootstrap method.

Now let's suppose that population distributios follows $N(\mu = 1.3, \, \sigma = 2)$


```R
sig = 2; mu = 1.3;
```

We will check the performance of the bootstrap method by changing the number of given sample to 50, 100, 500, or 5000. And we will generate 10000 bootstrap samples. Each bootstrap samples will have same number of samples with the given samples.


```R
num_samples = c(50, 100, 500, 5000)
num_B = 10000
```

Let's find estimated 95% confidence interval for $\mu = 1.3$   


```R
alpha = 0.05
res = c()

for (n in num_samples){
    # Given samples from the population distribution
    x = rnorm(n, mean = mu, sd = sig)
    
    # 95% Confidence interval that we can get if we know the population distribution. 
    # We want to estimate this confidence interval by bootstrap method.
    CI_lower = mean(x) - 1.96 * sig / sqrt(n)
    CI_upper = mean(x) + 1.96 * sig / sqrt(n)
    
    # Estimate confidence interval by bootstrap method
    boot_res = boot_mean_CI(data = x, B = num_B, alpha = alpha)
    res = rbind(res, c(CI_lower, CI_upper, 
                       boot_res$CI_lower, boot_res$CI_upper))
}

res = matrix(res, nrow = length(num_samples), ncol = 4)
dimnames(res) = list(num_samples, c("CI_lower", "CI_upper", "boot_CI_lower", "boot_CI_upper"))
res
```


<table>
<caption>A matrix: 4 × 4 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>CI_lower</th><th scope=col>CI_upper</th><th scope=col>boot_CI_lower</th><th scope=col>boot_CI_upper</th></tr>
</thead>
<tbody>
	<tr><th scope=row>50</th><td>0.5552793</td><td>1.664023</td><td>0.5586696</td><td>1.669612</td></tr>
	<tr><th scope=row>100</th><td>1.1076247</td><td>1.891625</td><td>1.1082261</td><td>1.894293</td></tr>
	<tr><th scope=row>500</th><td>1.0594518</td><td>1.410067</td><td>1.0571942</td><td>1.409875</td></tr>
	<tr><th scope=row>5000</th><td>1.1960689</td><td>1.306943</td><td>1.1962383</td><td>1.306547</td></tr>
</tbody>
</table>



- Confidence interval from the bootstrap method is more conservative. That is, have more broad range of the confidence interval.
- The more samples given (means $n \rightarrow \infty$), the better the quality of the estimated confidence interval. Almost same with the confidence interval calculated by infromation from the population distribution.

How stable is this bootstrap confidence interval? Let's check also the coverages. 

Now only consider 50 or 100 given samles. We will check the stability of the confidence interval from the bootstrap by doing 500 times.


```R
num_samples = c(50, 100)
n_iter = 500
```


```R
coverage_mean = c(); coverage_se = c()
alpha = 0.05

normal_boot_CI_lower = c()
normal_boot_CI_upper = c()

for (n in num_samples){
    is_covered = c()
    
    for (i in c(1:n_iter)){ # We will check the coverage of the bootstrap confidence interval    
                            # by sampling bootstrap samples 500 times
        x = rnorm(n, mean = mu, sd = sig)
        
        boot = boot_mean_CI(data = x, B = num_B, alpha = alpha)
        is_covered = c(is_covered, (boot$CI_lower <= mu) & (mu <= boot$CI_upper)) 
        
        normal_boot_CI_lower = c(normal_boot_CI_lower, boot$CI_lower)
        normal_boot_CI_upper = c(normal_boot_CI_upper, boot$CI_upper)
    }
    
    coverage_mean = c(coverage_mean, mean(is_covered))
    coverage_se = c(coverage_se, 
                    sqrt(alpha * (1 - alpha) / iter)) # Binomial standard error
}

res = matrix(c(coverage_mean, coverage_se), byrow = TRUE, nrow = 2, ncol = length(num_samples))
dimnames(res) = list(c("coverage", "Monte Carlo SE"), num_samples)
res
```


<table>
<caption>A matrix: 2 × 2 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>50</th><th scope=col>100</th></tr>
</thead>
<tbody>
	<tr><th scope=row>coverage</th><td>0.930000000</td><td>0.950000000</td></tr>
	<tr><th scope=row>Monte Carlo SE</th><td>0.009746794</td><td>0.009746794</td></tr>
</tbody>
</table>



- For both sample cases, bootstrap confidence interval covered $\mu$ almost 96% out of 500 times.
- Standard error of this coverage over 500times is almost 1%, very low for both sample cases.


```R
options(repr.plot.width = 15, repr.plot.height = 8)

normal_interval_values = data.frame(rbind(cbind(normal_boot_CI_lower, "lower_bound"),
                                    cbind(normal_boot_CI_upper, "upper_bound")))
colnames(normal_interval_values) = c("value", "category")
normal_interval_values$value = as.double(normal_interval_values$value)

mean_info = aggregate(normal_interval_values$value, 
                      list(normal_interval_values$category),
                      FUN = mean)

ggplot(normal_interval_values, aes(x = value, color = category)) + 
    geom_histogram(fill = "white", alpha = 0.5, position = "identity") + 
    geom_vline(data = mean_info, aes(xintercept = x, color = Group.1),
               linetype = "dashed") + 
    geom_vline(xintercept = 1.3, color = "black") + 
    labs(x = "Estimated CI value",
         y = "Count") +
    theme(axis.title.x = element_text(size = 20),
          axis.title.y = element_text(size = 20),
          axis.text.x = element_text(size = 15),
          axis.text.y = element_text(size = 15),
          legend.key.size = unit(2, "cm"),
          legend.text = element_text(size = 20),
          legend.title = element_text(size = 20))

```

    `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    



    
![png](/assets/images/coursework/STATS509/3.bootstrap_files/3.bootstrap_22_1.png)
    


- We can see that our estimated confidence intervals are near the population mean value $\mu = 1.3$ (black verticle line)

$\textbf{Example 2}$: Bootstrap fails!

What if we want to estimate the location parameter of the t-distribution?

Let's set a location parameter of the population t-distribution as 0.5, which is a rather heavy tailed t-model.   
Here $\mu$ is also set to 1.3 as above example.


```R
nu = 0.5
mu = 1.3
```

Now only consider 50 or 100 given samles. We will check the stability of the confidence interval from the bootstrap by doing 500 times.


```R
num_samples = c(50, 100)
n_iter = 500
```


```R
coverage_mean = c(); coverage_se = c()

t_boot_CI_lower = c()
t_boot_CI_upper = c()

for (n in num_samples){
    is_covered = c()
    
    for (i in c(1:n_iter)){ # We will check the coverage of the bootstrap confidence interval    
                            # by sampling bootstrap samples 500 times
        x = mu + rt(n, df = nu) # Given samples from t-distribution with mu = 1.3 and df = 0.5
        
        boot = boot_mean_CI(data = x, B = num_B, alpha = alpha)
        is_covered = c(is_covered, (boot$CI_lower <= mu) & (mu <= boot$CI_upper)) 
        
        t_boot_CI_lower = c(t_boot_CI_lower, boot$CI_lower)
        t_boot_CI_upper = c(t_boot_CI_upper, boot$CI_upper)
    }
    
    coverage_mean = c(coverage_mean, mean(is_covered))
    coverage_se = c(coverage_se, 
                    sqrt(alpha * (1 - alpha) / iter)) # Binomial standard error
    
    
}

res = matrix(c(coverage_mean, coverage_se), byrow = TRUE, nrow = 2, ncol = length(num_samples))
dimnames(res) = list(c("coverage", "Monte Carlo SE"), num_samples)
res
```


<table>
<caption>A matrix: 2 × 2 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>50</th><th scope=col>100</th></tr>
</thead>
<tbody>
	<tr><th scope=row>coverage</th><td>0.998000000</td><td>0.998000000</td></tr>
	<tr><th scope=row>Monte Carlo SE</th><td>0.009746794</td><td>0.009746794</td></tr>
</tbody>
</table>



- We can see that the confidence interval from the bootstrap method is failed because the range of confidence interval is too wide, which give us 100% coverage. 
- We can check this by below histograms of estimated confidence interval values.Estimated confidence interval values are located far away from the population mean value $\mu = 1.3$, which makes the coverage be almost 100%.


```R
options(repr.plot.width = 15, repr.plot.height = 8)

t_interval_values = data.frame(rbind(cbind(t_boot_CI_lower, "lower_bound"),
                                     cbind(t_boot_CI_upper, "upper_bound")))
colnames(t_interval_values) = c("value", "category")
t_interval_values$value = as.double(t_interval_values$value)

mean_info = aggregate(t_interval_values$value, 
                      list(t_interval_values$category),
                      FUN = mean)

ggplot(t_interval_values, aes(x = value, color = category)) + 
    geom_histogram(fill = "white", alpha = 0.5, position = "identity") + 
    geom_vline(data = mean_info, aes(xintercept = x, color = Group.1),
               linetype = "dashed") + 
    geom_vline(xintercept = 1.3, color = "black") + 
    labs(x = "Estimated CI value",
         y = "Count") +
    theme(axis.title.x = element_text(size = 20),
          axis.title.y = element_text(size = 20),
          axis.text.x = element_text(size = 15),
          axis.text.y = element_text(size = 15),
          legend.key.size = unit(2, "cm"),
          legend.text = element_text(size = 20),
          legend.title = element_text(size = 20))

```

    `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    



    
![png](/assets/images/coursework/STATS509/3.bootstrap_files/3.bootstrap_32_1.png)
    


## [ 3.2. When does bootstrap fail? : Theoretical argument]

Suppose $X_1, ..., X_n \sim Unif(0, \theta), \quad \theta \gneq 0$      
Let $\hat \theta = X_{(n)}$ be the maximum, which is the MLE. 

Then, note that 
$$
P_{*}(\hat \theta ^ {*} - \hat \theta = 0) = 1 - (\frac{n - 1}{n}) ^ n
$$

So, since 
$$
(\frac{n - 1}{n}) ^ n = (1 - \frac{1}{n}) ^ n \rightarrow e^{-1}, \quad as \quad n \rightarrow \infty \\
$$

this means that 
$$
P_{*}(\hat \theta ^ * - \hat \theta = 0) \rightarrow 1 - e^{-1}, \quad as \quad n \rightarrow \infty \\
$$

That is, the bootstrap is inconsistent, because 
$$
P(\hat \theta - \theta = 0) = 0, \quad \text{for all} \; n
$$

$<DEF: Consistency \; of \; the \; bootstrap>$        
: Suppose $T(X_1, ..., X_n, F)$ denotes a functional of the data $X_1, ..., X_n$ and a distribution $F$ generating the data. Think of $\sqrt{n}(\bar X_n - \mu) = T_{mean}(X_1, ..., X_n, F)$      

: Consider the distribution function  

$$
H_n(x) = P(T(X_1, ..., X_n, F) \leq x) \\
$$

$\,$ and its bootstrap version   

$$
H_{n, boot}(x) = P_{*}(T(X_{1}^{*}, ..., X_{n}^{*}, \hat F_{n}) \leq x) \\
$$

$\,$ where $P_{*}$ means that we treat the data as fixed and sample from $\hat F_{n}$ independently    

: Let $\rho$ be a metric measuring proximity between distribution functions. For example,   

$$
\rho (F, G) = \underset{x \in \mathbb{R}}{sup} |F(x) - G(x)|
$$  

$\,$ is the Kolmogorov-Smirnov metric.   
  
: Then we say that the boostrap is consistent for $T$ and $\rho$, if   

$$
\rho (H_{n, boot}, H_{n}) \rightarrow 0, \quad \text{in probability}
$$

$< THM : Bickel \, and \, Freedman>$    
: If $X_1, ..., X_n \sim F$ are iid and $\mathbb{E}(X_{1}^{2}) \lneq \infty$,     
$\,$ then for $T_{mean}(x_1, ..., X_n, F) = \sqrt{n}(\bar X_n - \mu)$, the bootstrap is consistent in the Kolmogorov and Wasserstein sense.

When does bootstrap fail?    
: Generally it fails to be consistent for functionals $T$ that do not satisfy the CLT. This is the case for heavy tailed data.

$\textbf{How to fix the bootstrap when it doesn't work?}$

m-out-of-n boostrap:     
- Resampling n times from a sample $X_1, ..., X_n$ of size $n$ is just too much.     
- Let $m(n) = o(n)$ and consider the independent boostrap samples    
   
$$
X_{b, 1} ^{*}, ..., X_{b, m}^{*}, \quad b = 1, ..., B \\
$$     

- Then, build statistics $\hat \theta_{b}^{*}$, $\quad b = 1, ..., B$   
- Since $m(n) = o(n)$, the CDF $F_n$ converges to the true $F$ at a faster rate thatn te statistics $\hat \theta^*$ converges to $\hat \theta$.    
- So, if $\sqrt{n}(\hat \theta - \theta) \rightarrow N(0, \sigma^2)$, in many cases we continue to have   

$$
\sqrt{m}(\hat \theta^* - \hat \theta) \rightarrow N(0, \sigma ^ 2)
$$

## [ 3.3. Application of the bootstrap: Bias, Variance and MSE approximation]

If we have consistency, and in many cases we do, then using the idea that      
   
$$
F_n(x) \approx F(x) \\   
$$   

we apply the ordinary boostrap to a wide variety of problems.

- Bootstrap approximation of the bias:   

$$
Bias(\hat \theta) = \mathbb{E}(\hat \theta - \theta)\approx \bar {\hat \theta ^ * } - \hat \theta = Bias_{boot}(\hat \theta)
$$

- Bootstrap approximation of the variance:    
   
$$
\sigma_{boot}^{2}(\hat \theta) = \frac{1}{B}\sum_{b = 1}^{B}(\hat \theta_{b}^{*} - \bar {\hat \theta^{*}}) ^ 2
$$

- Which gives    
   
$$
MSE_{boot} = Bias_{boot}^{2}(\hat \theta) + \sigma_{boot}^{2}(\hat \theta)
$$
