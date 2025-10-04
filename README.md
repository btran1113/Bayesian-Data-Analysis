# Bayesian-Data-Analysis

## Summary:
Extracted generated dataset from the U.S. Census Bureau on 1338 subjects to evaluate the correlation of smoking and medical costs. Using R, conducted MCMC diagnostics on a Bayesian GLM. Used cross-validation to compare posterior predictive quality of log transformed models. Compiled results, summary tables, and visualizations. There are 1338 subjects possessing variables that reflect health like BMI and smoking status, demographic information like age, sex, and residential region of the US, and number of children covered by the insurance. These predictors can be used on the response variable, medical expenses billed. 

<img width="600" height="513" alt="image" src="https://github.com/user-attachments/assets/24e38c96-9778-4166-adb4-0b22e8856ca8" />

We observe a generally positive correlation between age and medical expenses, as well as smokers having higher costs on average than non-smokers. This can be explained by smoking being a risk factor to preventable diseases such as lung cancer, heart disease, stroke, and more.

## Dataset used:
https://www.kaggle.com/datasets/mirichoi0218/insurance
