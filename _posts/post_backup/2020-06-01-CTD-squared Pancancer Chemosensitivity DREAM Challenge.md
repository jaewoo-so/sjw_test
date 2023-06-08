---
layout: post
title:  "CTD-squared Pancancer Chemosensitivity DREAM Challenge"
#tags: [competitions]
---

# Gene Expression Searching and machine learning model for Chemosensitivity Prediction
---


***link***    
[official competetion site]()    
[github-repository]()    


***contents***
1. Data
2. Workflow & Key ideas
3. Conclusion & Discussion
4. Key Idea & Feature Engineering
5. Reference

- If you want to see entire predictive system concept, 
see section 2
- If you want to detail of machine learning strategies, see section 5
- The data used in this work are all public data.

- In the middle, we changed the working environment to the AWS environment. The code for this repository is still unorganized, so it can be messy. And because the data is very large, I didn't put it in the repository.


&nbsp;
&nbsp;
# 1. Data

### **Used Dataset**
- CCLE basal expression & meta info
- L1000 phase1, phase2 lv.5
- CTRP AUC
- DEMETER2 normalized dependency score for 515 cell lines
- PANACEA gene expression

### **Preprocessing**

Use the following five data to describe the characteristics of the cell line.
Histology and basal expressions for 515 cell lines in CCLE ,
TF activity inference score and Pathway inference score were calculated from the CCLE basal expression value using Viper. PROGENy, respectively.
The NA values within the DEMETER2 score were imputed with average values per cell lines.

For the data to describe the characteristics of the drug, only post-treatment expression values were used.
Signatures for overlapping 326 drugs in CTRP and L1000 are selected and only 973 experimentally measured genes were used to normalize by MODZ.
The given PANACEA expression values were also normalized by the MODZ method across the cell lines.

$$\mathcal{D_f} = \{  (x_i,y_i) \vert  f \in F\}$$

where $y$ is auc value of perturbation, $x_i$ is feature of each sample. 

$\mathcal{S}$ is feature class, $\mathcal{S} = \{TF,PRO,HIST,GENE,CCLE,D2_{mean} \}$, $\mathcal{F}$ is  feature set, $\mathcal{F} = \{f_i \vert  i \in S \}$

$$f^* = \{ TF,HIST,GENE,CCLE,D2_{mean} \}$$

&nbsp;
&nbsp;
# 2. Workflow & Key ideas

&nbsp;
### Step1. Searching similar drugs for hidden drugs
---
First, we performed the gene expression signature search via spearman correlation calculation. 
As a reference data, L1000, which was signaturized for each drug through MODZ, and PANACEA was used as a query for this. The similarity between drugs was calculated by spearman correlation. We chose similar drugs with the following criteria. 
By calculating robust z-score for the similarity matrix, only drugs corresponding to more than 70% of the max value of z-score were collected for each query drug. For example, for the above method, there are 7 and 19 drugs selected as drugs similar to cmpd_KW and cmpd_WW, respectively.

<p align="center">
    <img width="400" src="./img/z_score_thereshold.png">
</p>


&nbsp;
### Step2. Modeling & Prediction
---
As shown above, a drug-specific model was created using only the selected perturbations for each hidden drug.
The model approached the regression problem using the GBDT to predict the AUC value according to the cell line-drug pairs given in the CTRP and used features describing the above mentioned. 
We use CART for base learner with histogram optimized approximate greedy algorithm.  We finally found out that roughly 3000 features exhibited the best performance, and we actually saw that performance was not significantly reduced even if we reduced to 7 feature sets through dimension reduction.  The most efficient dimensional reduction model was the Autoencoder model with MSE + Correlation*0.7 as the objective, but due to the complicated analysis, we use the gradient boosting method with the CART for base learner and use a very small number of sampled features. And by overfitting each prediction model to a specific set of drugs,  final ensemble model had better generalization performance since the diversity of models increased. 
<p align="center">
    <img width="800" src="./img/structure.png" alt="Material Bread logo">
</p>

&nbsp;
### Step3. Ensemble
For generalization performance, we implemented a model ensemble. Using the results of varying the threshold value from 20% to 90% of max with z-score as shown above, correlation between the results of each model was compared. 
To use more diverse models, eight models were finally selected in the order in which they were less correlated with each other, and the predicted results were integrated to create a final presentation file.
Detail procedure is like this.

$$ \mathcal{M} \textrm{ is set of models,} \mathcal{M} := \{ M_1 , \cdots , M_n \}, \mathcal{D} : \mathcal{M} \times  \mathcal{M} \rightarrow \mathcal{R} \textrm{ is distance function}$$ 
and we define average distance over each model. 

$$ \mathcal{D}_{avg}(M_i) = \frac{1}{| \mathcal{M} \setminus { \{i\} }  |}  \sum_{m \in \mathcal{M} \backslash {\{i\}} } \mathcal{D}(M_i,m)$$ 

next, we select candidate for ensemble .

$$\mathcal{M}_{candidate} :=\{  \mathcal{M}_i | \mathcal{D}_{avg}(M_i) \ngeq \textrm{ 90th percentile} \}$$

finaly, define of ensemble score is like below .

$$ S_{ensemble} = exp( \frac{1}{|\mathcal{M}_{candidate}|}  \sum_{m \in \mathcal{M}_{candidate}} log( \mathcal{D}_{avg}(m)) ) $$

&nbsp;
&nbsp;
# 3. Conclusion & Discussion
We started from the assumption that we could deduce the MoA of the drug via post-treatment gene expression on the drug.
It is difficult to predict drug sensitivities that have specific targets and specific pathways(Koras, K et al. 2020). So, we recognized the need to make models for each drug, and we tried many things, such as looking at which gene has high coefficient for each drug by predicting AUC with only gene expression, in addition to the methods described above. Our experiments have also shown that it is better to make a model for each drugs than to use one model predicting the AUC for all drugs.
Each cell line has its own unique characteristics, so it is very meaningful to use it as a feature that can explain it. Predicting the drug sensitivities for each cell line will be very useful in selecting cell lines in experimental screening and also valuable in the field of new drug development in reducing costs.
Since we didn't have enough time to find the optimal model, we had no choice but to use the naive experimental results, so I think we can improve the predictive performance through more accurate experiments.

&nbsp;
&nbsp;
       


&nbsp;
&nbsp;
# 4. Key Idea & Feature Engineering

**1. Data Integration**     
Different public datasets have different gene types. Alternatively, you can use a gene set that you deem valid based on domain knowledge. Strategies other than intersection result in missing value unconditionally. Therefore, the method of imputating the missing value must also be selected.       
 In this project, we compared two methods using gene sets judged to be valid through intersection and paper search, and found that the performance of intersection is similar. Therefore, we used an intersection geneset with a small data size. (973 genes.)

**2. Demesional reduction**     

- Unable to find any studies for manifolds of data between total expression amount + chemical reaction amount. Therefore, Autoencoder was used.

- Use cosine similarity and pearsons R as the evaluation metric. In a situation where the process of generating genetic data is not guaranteed to be the same, it is difficult to normalize when operating between different dataset. Therefore, Cosine similarity was used as the main and Pearson correlation coefficient was used as an adjunct.

- VAE vs AE comparison. (see '/code_clean/st_04_mapping_drug_378norm_ctrp_auto_encoder/')
<p align="center">
    <img width="600" src="./img/vae_ae.png" alt="Material Bread logo">
</p>

&nbsp;

**Why perform demesional reduction and use cosine similarity for AE?**    
-> I found that the genomic data we use is biased by sequencing machines and processes in training machine learning models. This was previously based on cancer tumor classification tests and genomic data.    
&nbsp;
These differences mainly occur in relative expression amounts, and it was experimentally found that the types of genes expressed are generally consistent.    
&nbsp;
This is why I used AE and cosine similarity as an evaluation metric.


&nbsp;       
**Why not UMAP(Uniform Manifold Approximation and Projection)?**  
 -> In the case of UMAP, it is necessary to design a quality evaluation method of the pre-distance metric, search for the optimal metric, and optimize the projection parameters. And since the model description is not necessary for this competition, it was not used for the sake of time.       
      
     
&nbsp;    
**Why not linear dimensionality reduction (like PCA, NMF)?**    
 -> Because the data is high-dimensional and sparse, the linear method does not fit.

&nbsp;    
**3. DNN encoder feature**      
-> It is a method determined in the engineering process to create optimal features.    
&nbsp;
Experimentally tried several feature engineering and applied them because we achieved the best CV-score.


&nbsp; 
&nbsp; 

# 4. References
1. CTD-squared Pancancer Chemosensitivity DREAM Challenge (syn21763589)     
2. CTD-squared BeatAML DREAM Challenge (syn20940518)        
3. Rees, M., Seashore-Ludlow, B., Cheah, J., Adams, D., Price, E., Gill, S., Javaid, S., Coletti, M., Jones, V., Bodycombe, N., Soule, C., Alexander, B., Li, A., Montgomery, P., Kotz, J., Hon, C., Munoz, B., Liefeld, T., Dančík, V., Haber, D., Clish, C., Bittker, J., Palmer, M., Wagner, B., Clemons, P., Shamji, A., Schreiber, S. (2016). Correlating chemical sensitivity and basal gene expression reveals mechanism of action Nature Chemical Biology  12(2), 109-116. https://dx.doi.org/10.1038/nchembio.1986     
4. Subramanian, A., Narayan, R., Corsello, S., Peck, D., Natoli, T., Lu, X., Gould, J., Davis, J., Tubelli, A., Asiedu, J., Lahr, D., Hirschman, J., Liu, Z., Donahue, M., Julian, B., Khan, M., Wadden, D., Smith, I., Lam, D., Liberzon, A., Toder, C., Bagul, M., Orzechowski, M., Enache, O., Piccioni, F., Johnson, S., Lyons, N., Berger, A., Shamji, A., Brooks, A., Vrcic, A., Flynn, C., Rosains, J., Takeda, D., Hu, R., Davison, D., Lamb, J., Ardlie, K., Hogstrom, L., Greenside, P., Gray, N., Clemons, P., Silver, S., Wu, X., Zhao, W., Read-Button, W., Wu, X., Haggarty, S., Ronco, L., Boehm, J., Schreiber, S., Doench, J., Bittker, J., Root, D., Wong, B., Golub, T. (2017). A Next Generation Connectivity Map: L1000 Platform and the First 1,000,000 Profiles Cell 171(6), 1437 1452.e17. https://dx.doi.org/10.1016/j.cell.2017.10.049        
5. Szalai, B., Subramanian, V., Holland, C., Alföldi, R., Pusk, L., Saez-Rodriguez, J. (2019). Signatures of cell death and proliferation in perturbation transcriptomics data—from confounding factor to effective prediction Nucleic Acids Research  47(19), 10010-10026. https://dx.doi.org/10.1093/nar/gkz805      
6. Koras, K., Juraeva, D., Kreis, J., Mazur, J., Staub, E., Szczurek, E. (2020). Feature selection strategies for drug sensitivity prediction Scientific Reports 10(1), 9377. https://dx.doi.org/10.1038/s41598-020-65927-9     
7. Garcia-Alonso L, Holland C, Ibrahim M, Turei D, Saez-Rodriguez J (2019). “Benchmark and integration of resources for the estimation of human transcription factor activities.” Genome Research. doi: 10.1101/gr.240663.118.      
8. Schubert M, Klinger B, Klünemann M, Sieber A, Uhlitz F, Sauer S, Garnett MJ, Blüthgen N, Saez-Rodriguez J. “Perturbation-response genes reveal signaling footprints in cancer gene expression.” Nature Communications: 10.1038/s41467-017-02391-6 