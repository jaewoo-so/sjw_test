---
layout: post
title:  "Gimhae Fire Prediction Competition"
tags: [competitions]
---

please go to below link for code and detail explain.
Code and data available.
[go to repository]([/assets/2020/01_23/canberra.png](https://github.com/jaewoo-so/gimhae_fire_prediction))


# 01 EDA and Basic Feature engineering 
---
[see jupyter notebook : 01 EDA and Basic Feature engineering  ](https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/01_EDA_Missing_Value_Basic_Processing.ipynb)

-  Basic feature preprocessing using EDA and domain knowledge

First, you need to check whether the data treated as NaN are missing or sparse.
Then, the type of data is checked, and pre-processing and feature engineering are performed accordingly.

Find answers to the questions below here.     
A. Missing or Sparse?    
B. What type of Missing or Sparse?    
C. If values type is missing, is there any meaning of missing?    


# 02 Experimental Feature engineering 
[see jupyter notebook : 02 Experimental Feature engineering ](https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/01_EDA_Missing_Value_Basic_Processing.ipynb)

- Feature engineering based on Cross Validation score and Test score. 


## Base on Cross Validation Score
### 01 convert missing value to min value : 강수량 prcpttn ()
- In the feature of precipitation, NaN is judged to be unmeasurable because there is no precipitation. change to minimum value 0

### 02 Fill features of categorical and binary type with the most frequent values 
- Due to the time relationship, the imputation model is not trained separately, and is processed as the most frequent value    

### 03 Numeric type feature missing value handling
- Processing based on domain knowledge    

**to zero**
ttl_grnd_flr : Sum of above-ground floors of buildings      
ttl_dwn_fr : Sum of basement floors of buildings     

**fill with mean value**
Since the following features have a low null_ratio, better results can be obtained by imputation by creating k-mean or a separate prediction model.    
As a matter of time, after imputation as an average, trials of other methods were followed.    

tmprtr : temperature (c)
wnd_spd : wind speed
wnd_drctn : wind direction
hmdt : humidity
hm_cnt : Administrative district Population
bldng_ar : Building area
fr_mn_cnt : Personnel of the fire department in charge

### 04 Target encoding with smoothing
- Experimentation with valid encoding methods
1. target encoding
2. target encoding + smoothing
3. target encoding + noise (0.1)
4. target encoding + noise (0.4)
5. target encoding + smoothing + noise (0.1)
6. target encoding + smoothing + noise (0.4)


### 05 Electrocity and Gas usage
- When looking at the data distribution, there are samples with no usage.     
- Assuming that this would be an empty house or an unmanaged building, I created the feature as shown below.    
- In the case of a building without electricity, it is assumed that the risk of fire will be high due to lack of management.    
- Fire has nothing to do with usage, so it is expressed in binary       

ele_engry_us : Electricity usage for a specific period    
gas_engry_us : Gas usage for a specific period    

  
1 -> A house that is being used or inhabited    
0 -> A building that is obsolete.    

In 01_EDA_Missing_Value.ipynb, check that min value = 0 of electricity and gas consumption data.    

### fr_mn_cnt: Number of fire department personnel

The number of fire department personnel in the jurisdiction is limited to a set    
Assume that fire department personnel are assigned according to the size of the area and the frequency of fire occurrence.    
Set Binary to 1 if more than 100, 0 if less than 100    

## Based on test score 

### Multiple encoding tests
[see jupyter notebook : experiment result Lightgbm](https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_exp/sojaewoo/data_1111_version/code_v07/001_1_encoding_evaluation_lgb.ipynb)    
[see jupyter notebook : experiment result Xgboost](https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_exp/sojaewoo/data_1111_version/code_v07/001_1_encoding_evaluation_xgb.ipynb)    


### wnd_drctn (0.08 score up)
<p align="center">
  <img src="https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/resource_data/wind.png">
</p>
- Assuming that the wind direction is different for each season, there is a possibility of fire depending on the direction of the season    
- Data categorization into 4 types according to wind direction    
- Apply binning to wind direction (wnd_drctn)     



## bin_hour : created feature (0.03 score up)
<p align="center">
  <img src="https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/resource_data/fire.png">
</p>
- Assume that it will be difficult to report a fire at dawn.    
- As a result of the check, according to the statistics of Seoul, there is a difference in the occurrence of fires according to time zones.    
- Similarly, Gimhae Fire & Marine Insurance determined that there was a difference in the frequency of fire occurrence by time period, and categorized data by time period.      



# 03 Data distribution control

- Check the distribution of Train / Validation / Test data. Control the distribution similarly to the actual test data to be used.    
- What is unusual about this competition is that validation data was given. Therefore, it was assumed that at least the validation data would have a similar distribution to the test data, but the difference and correlation between the CV score and the LB score could not be derived, so the work was performed assuming that the distribution would be different.    

- Adverserial validation 및 UMAP dimension reduction visualization 으로 확인 
    - [see jupyter notebook : Adverserial Validation](https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/etc_experiment/adverserial_validation/#01_adverserial_test_v11.ipynb)    
    - [see jupyter notebook : UMAP dimension reduction visualization](https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/etc_experiment/umap_dimentional_reduction/01_datav04_Apply_umap_test.ipynb)    


# 04 Sampling
- The following sampling strategies were used to control for differences in data distribution.    

## 1. A dimensionally reduced, low-dimensional vector that samples only samples that are close to the test sample.    

 
<p align="center">
  <img src="https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/resource_data/euclidian.png">
</p>
euclidean distance    
    

The black dot is the test data. Model training proceeds by removing samples that are far from the black point distance and sampling only samples that are close to the black point.
The criterion of closeness was experimentally determined.    


## 2. Picking with Predicted Probabilities    
- Create a train or not, validation or not, test or not classification model and extract train and validation samples that are indistinguishable from test.    

<p align="center">
  <img src="https://github.com/jaewoo-so/gimhae_fire_prediction/blob/master/code_final/resource_data/te.png">
</p>
Probability distribution of predicted values for test data or not.    
Even though it is test data, there are also samples that are not test data. At this time, select training and validation samples that are predicted with a similar probability to the test data. (mainly between 0.05 and 0.15)    
Conversely, samples predicted as test data are also selected even though they are not test data. (mainly between 0.8 and 1.0)    
