---
layout: post
title:  "Clinical decision support algorithm based on machine learning to assess the clinical response to anti–pd-1 therapy"
#tags: [publication_patents]
---


## A tissue origin prediction device, method of predicting the tissue origin using a genome data, and computer program
### 1020200076756 · Filed Jun 23, 2020

# Summary 
---

 Anti-programmed death (PD)-1 therapy (αPD-1) has been used in patients with non-small cell
lung cancer (NSCLC), leading to improved outcomes. However, the outcomes of therapy are still
insufficient, and the expression of PD-ligand 1 (PD-L1) is not always a predictor of response to
αPD-1. 

 NSCLC의 치료 목적으로 사용되는 면역 항암제는 일반적으로 키트루다와 옵티보이다.
한 달 치료비는 각각 약 60만 달러와 80만 달러이며, 연간 거의 천만 달러의 비용이 든다.
 The immuno-cancer drugs used for treatment purposes of NSCLC are typically Kitruda and Optivo.
Each monthly treatment costs about $600,000 and $800,000, respectively, and costs nearly $10 million a year.


하지만 모든 환자에게 약물이 반응하지 않는다. 따라서 키트루다의 경우 PD-L1 발현 양성 비율'이 50% 이상이어야 처방을 받을 수 있다.하지만 조건이 만족되어 처방을 받아도 실제 약물반응은 약 60% 대 이다. 
But the drug doesn't respond to all patients. Therefore, in the case of Kitruda, the 'PD-L1 expression positive rate' must be 50% or more to be prescribed. However, even if the conditions are satisfied and prescribed, the actual drug reaction is about 60%.

우리는 다른 환자의 정보를 사용하여 약물 예측 모델을 연구하여 좀더 많은 환자들이 올바른 처방을 받고 약물 낭비에 의한 비용을 줄이려 한다. 

# Summary Idea & Solution
---

## 사용된 데이터 
- clinical data including patient characteristics, 
- mutations
- laboratory findings from the electronic medical records

## 데이터 
- missing value가 대부분의 피쳐에 존재. 단순 mean, zero 등의 방법이 아닌, 다른 피쳐 및 해당 피쳐의 non-missing value의 분포를 기준으로 inputation 모델을 별로도 학습. 
- Data leakage가 발생하지 않도록, 임상의사에게 직접 검토를 받았다. 

## Feature Enegineering 
- 유전자 데이터의 경우 Sparse 특성이 있었다. -> Binary 형태의 관련성 있는 유전자 발현 여부를 나타내는 새로운 Feature 생성
- Neutrophil 및 Lymphocyte의 경우는 비율인 LNR(lymphocyte-to-neutrophil ratio) 사용 

## 모델 선택 및 검증
- 모델의 적절한 복잡성을 찾기 위해 리니어 모델도 포함하여 모델 선택 진행. 
- 샘플 수가 적어서 모델 선택에는 LOOCV 사용
- 분야 특성상 어느정도의 설명력이 필요하여, 성능 극대화를 위한 앙상블 단계를 진행하지 않음. 
- LOOCV 결과로 어느정도 아웃라이어로 보이는 샘플들을 발견. (4개의 샘플)

## 모델 기반 피쳐 분석
- 데이터간의 interaction 효과가 있을 가능성이 매우 높다. 
- 따라서 단순 entropy기반의 피쳐 중요도는 해석에 오류가 생길 가능성이 높음.   
- Lime 기반의 explanation model은 샘플수가 적어서, local model 피팅에 문제가 생길 여지가 많다고 판단.  
- 각 샘플 대상으로 shap value의 평균 및 interaction value를 계산했다. 


![main_fig](/assets/publication_patents/CDSS_main.jpg)
![front](/assets/publication_patents/paper_front.png)


# Detail 
---
## Feature engineering 

### Gene& Metastasis Feature 

**Sparsity**
일반적으로 많이 사용되는 sparsity 판단 기준으로 판단시, zero value의 비율에 의한 sparsity가 있다고 판단되었다.
- Percentage of zero values : 데이터 타입이 발현 유무의 Bianry 타입 \$ Gene \ {0,1}\$. 최소 85% ~ 최대 95%의 zero value가 보인다. 따라서 sparse하다고 판단. 
- Number of observations : observations에 의한 sparse는 보이지 않는다. 
- Variability of data : 바이너리이기 때문에 Percenrage of zero 와 같은 결론이 나온다. Sparse하다 
- Model performance: 샘플 수의 한계로 판단이 불가 
- Domain knowledge : 대한 유전자 발현에 대한 정확한 통계가 없으므로, Domain knowledge 기준으로 sparsity를 판단 할 수 없다. 

**New Feature**
- Sparsity를 감소키기위해서 유전자 피쳐를 관련 유전자의 총 발현량 수를 나타내는 새로운 피쳐로 변경 
$$\text{driver oncogene} = \sum{ \text{gene expression}}$$
- Metastasis 도 같은 방식으로 총 Metastasis로 변경 
$$\text{metastasis count} = \sum{ \text{metastasis present}}$$
- Neutrophil 및 Lymphocyte은 LNR(lymphocyte-to-neutrophil ratio) 피쳐로 대체했다. 
$$LNR = log(\frac{Neutrophil}{Lymphocyte} + \epsilon )$$


# Model Selection
---
아래는 8개의 모델의 roc-auc score 비교 표 이다. LOOCV (Leave one out CV)로 검증을 하였다. 
예측에 필요한 모델은 높게 복잡하지 않아도 되는것으로 판단되었다. 따라서 모델 앙상블은 하지 않고 싱글 모델만 비교했다. 

![score](/assets/publication_patents/paper_compare.png)

<br/>
    
### Best Model Performance
![score](/assets/publication_patents/paper_score.png)



    
# Feature Attribution
---
모든 샘플의 SHAP value 평균값이다. 
Shap value는 게임이론 관점에서, 한 피쳐의 및 다른 피쳐와의 시너지 효과까지 고려한 모델의 예측 값에 대한 기여도이다. 
의료, 바이오 분야의 데이터는 독립성을 주장하기 어려운 피쳐들이기 때문에 SHAP 값 기반의 해석이 적절하다고 판단했다. 
![shap](/assets/publication_patents/paper_shap_val.png)

결과를 보면 The persence of non-measurable lesions가 타겟값과의 correlation이 높을 수도 있다는 의심을 했다. 따라서 
The persence of non-measurable lesions 의 경우 오디너리 형식의 데이터이므로, spearman correlation을 살펴본다. 
correlation = -0.44 , p-value = 8.1e-11 이 나왔다. 즉, strong relationship이 아니므로 feature로 사용하였다.

<!-- 반응 여부 : 바이너리, lesions은 오디너리 형식(0,0.5,1)  
correlation을 스피어만 , 피어슨중 골라야함. 
설명을 하면 
스피어만은 monotonic relationship, 피어슨은 linear relationship 이 있는지를 보는 것이다. 
모노토닉은 한 변수가 증가할때 다른 변수도 증가하는지 여부만 보는것. 
linear는 한변수가 1증가시 다른변수도 같은양인 1로 증가하는지, 증가 폭도 같이 보는것이다. (양적관계도 포함되어있다.)
 -->

