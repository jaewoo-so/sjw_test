---
layout: post
title:  "Clinical decision support algorithm based on machine learning to assess the clinical response to anti–pd-1 therapy"
tags: [publication_patents]
---


## A tissue origin prediction device, method of predicting the tissue origin using a genome data, and computer program
### 1020200076756 · Filed Jun 23, 2020

# Summary 
---

 Anti-programmed death (PD)-1 therapy (αPD-1) has been used in patients with non-small cell
lung cancer (NSCLC), leading to improved outcomes. However, the outcomes of therapy are still
insufficient, and the expression of PD-ligand 1 (αPD-L1) is not always a predictor of response to
αPD-1.   

 The immuno-cancer drugs used for treatment purposes of NSCLC are typically Kitruda and Optivo.   
Each monthly treatment costs about $600,000 and $800,000, respectively, and costs nearly $10 million a year.   

But the drug doesn't respond to all patients. Therefore, in the case of Kitruda, the 'PD-L1 expression positive rate' must be 50% or more to be prescribed. However, even if the conditions are satisfied and prescribed, the actual drug reaction is about 60%.   

I studied a drug activity prediction model using Non-FDA-approved information to help more patients get the correct prescription and reduce the cost of drug waste.    


# Summary Idea & Solution
---

## Data 
- Clinical data including patient characteristics 
- Mutations
- Laboratory findings from the electronic medical records

## Data Feature 
- Missing values ​​exist in most features. Learning inputation models separately based on the distribution of other features and their non-missing values, rather than simple mean, zero, etc. methods.
- To prevent data leakage, it was reviewed directly by the clinician.

## Model Selection & Evaluation
- Proceed with model selection, including linear models, to find the appropriate complexity of the model.
- Due to the small number of samples, LOOCV is used for model selection
- Due to the nature of the field, some explanatory power is required, so the ensemble stage to maximize performance is not performed.
- Finding samples that appear to be somewhat outliers as a result of LOOCV. (4 samples)

## Feature Attribution
- It is very likely that there is an interaction effect between data.
- Therefore, feature importance based on simple entropy is highly likely to cause errors in interpretation.
- The Lime-based explanation model has a small number of samples, so it is judged that there is a lot of room for problems in fitting the local model.
- For each sample, the average of the shap values ​​and the interaction value were calculated.

![main_fig](/assets/publication_patents/CDSS_main.jpg)
![front](/assets/publication_patents/paper_front.png)


# Detail 
---
## Feature engineering 

### Gene& Metastasis Feature 

**Sparsity** 
<br>
 
When judging based on the commonly used sparsity judgment criteria, it was determined that there was sparsity by the ratio of zero value.
- Percentage of zero values: About $Gene Expression  \in \{0,1\}$,  The ratio of zero is at least 85% to a maximum of 95%. Therefore, it is judged to be sparse.
- Number of observations: It does not see the sparse by observices.
- Data Volatility: Because it is a binary, the conclusion is the same as that of Percenrage of zero. it is sparse.
- Model performance: Unable to judge due to the limit of the number of samples.
- Domain knowledge: Since there are no accurate statistics on gene expression for Korean, sparsity cannot be determined based on Domain knowledge.


**New Feature**  

- To reduce sparsity, change the gene feature to a new feature that represents the total number of expressions of the associated gene.
$$\text{driver oncogene} = \sum{ \text{gene expression}}$$
- Change Metastasis to "Total Metastasis" in the same way.
$$\text{metastasis count} = \sum{ \text{metastasis present}}$$
- Neutrophil and Lymphocyte were replaced by the LNR (lymphocyte-to-neutrophil ratio) feature.
$$LNR = log(\frac{Neutrophil}{Lymphocyte} + \epsilon )$$


<br>

## Model Selection
Below is a comparison table of the eight models of the roc-auc score. It was evaluated with LOOCV (Leave one out CV).
It was judged that the model required for prediction did not have to be highly complex. Therefore, we did not do model ensembles and compared only single models.  
![score](/assets/paper_cdss/paper_compare.png)
<br/>

**Best Model Performance : LightGBM**

![score](/assets/paper_cdss/paper_score.png)

<br/>

## Feature Attribution
---


It is the average value of the SHAP value of all samples.  
Shap is the contribution to the predicted value of a model considering the synergistic effect of one feature and another feature, from the point of view of game theory.   
Since the data in the medical and bio fields are features that are difficult to assert independence, it was judged that SHAP value-based interpretation was appropriate.    

![shap](/assets/paper_cdss/paper_shap_val.png)


Looking at the results, I suspected that the persistence of non-measurable lesions might have a high correlation with the target value. Thus, the persistence of non-measurable lesions is an ordinary type of data, so we look at spearman correlation.
correlation = -0.44 and p-value = 8.1e-11. That is, it was not a strong relationship, so it was used as a feature.





