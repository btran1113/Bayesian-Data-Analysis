```r
#Load packages
library(ggplot2)
library(rstanarm)
library(bayesplot)
library(bayesrules)
library(tidyverse)
library(tidybayes)
library(broom.mixed)
```
```r
#Read in dataset
library(readr)
health_insurance <- read.csv("health_insurance.csv", stringsAsFactors = TRUE)
```
```r
#Plot of observed relationship between medical expense, age, and smoking:
ggplot(health_insurance, aes(x = age, y = charges, color = smoker)) +
  geom_point() + 
  geom_smooth(method = "lm", se = FALSE)
```
```r
#Default rstanarm priors with the assumption that average charges tend to be around 20000:
health_main <- stan_glm(
  charges ~ age + smoker,
  data = health_insurance, family = gaussian, 
  prior_intercept = normal(20000, 2.5, autoscale = TRUE),
  prior = normal(0, 2.5, autoscale = TRUE), 
  prior_aux = exponential(1, autoscale = TRUE),
  chains = 4, iter = 5000*2,, seed = 84735)
```
```r
#MCMC diagnostics
mcmc_trace(health_main, size = 0.1)
mcmc_acf(health_main)
mcmc_dens_overlay(health_main)
neff_ratio(health_main)
rhat(health_main)
```
```r
#Posterior summary statistics
tidy(health_main, effects = c("fixed", "aux"),
     conf.int = TRUE, conf.level = 0.80)
```
```r
#Posterior models of typical medical charges for smokers and non-smokers
as.data.frame(health_main) %>% 
  mutate(no = `(Intercept)`, 
         yes = `(Intercept)` + smokeryes) %>% 
  mcmc_areas(pars = c("no", "yes"))
```
```r
#Prediction for 40 year old smoker with main model
set.seed(84735)
charges_prediction <- posterior_predict(
  health_main,
  newdata = data.frame(age = c(40), 
                       smoker = c("yes")))
mcmc_areas(charges_prediction) +
  xlab("charges") +
  ggplot2::scale_y_discrete(labels = c("Smoker"))
```
```r
#Visual posterior predictive check of main model
pp_check(health_main) +
  xlab("charges") +
  ylab("density")
```
```r
#Take log transform of Y to improve model by addressing violation of assumptions (prior intercept was selected by looking at range of logcharges in health_insurance dataset)
health_insurance$logcharges=log(health_insurance$charges)
health_transform <- stan_glm(
  logcharges ~ age + smoker,
  data = health_insurance, family = gaussian, 
  prior_intercept = normal(8.5, 0.75),
  prior = normal(0, 2.5, autoscale = TRUE), 
  prior_aux = exponential(1, autoscale = TRUE),
  chains = 4, iter = 5000*2,, seed = 84735)
```
```r
#Transformed model with interaction between age and smoking status
health_interaction <- stan_glm(
  logcharges ~ smoker + age + smoker:age,
  data = health_insurance, family = gaussian, 
  prior_intercept = normal(8.5, 0.75),
  prior = normal(0, 2.5, autoscale = TRUE), 
  prior_aux = exponential(1, autoscale = TRUE),
  chains = 4, iter = 5000*2,, seed = 84735)
```
```r
#Prediction for 40 year old smoker with interaction model
set.seed(84735)
charges_prediction2 <- posterior_predict(
  health_interaction,
  newdata = data.frame(age = c(40), 
                       smoker = c("yes")))
mcmc_areas(charges_prediction2) +
  xlab("logcharges") +
  ggplot2::scale_y_discrete(labels = c("Smoker"))
```
```r
#Visual posterior predictive check of transformed model with interaction term
pp_check(health_interaction) +
  xlab("log(charges)") +
  ylab("density")
```
```r
#10-fold cross-validation to compare the posterior predictive quality of both transformed models
set.seed(84735)
prediction_summary_cv(model = health_transform, data = health_insurance, k = 10)
```
```r
#Evaluate and compare the expected log-predictive densities (ELPD) posterior predictive accuracy of both transformed models
set.seed(84735)
loo_1 <- loo(health_transform)
loo_2 <- loo(health_interaction)
c(loo_1$estimates[1], loo_2$estimates[1])

loo_compare(loo_1, loo_2)
```
