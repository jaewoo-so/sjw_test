---
layout: post
title:  "ESG Media Topic Classification"
date:   2023-01-01
#tags: [project]
---

**Model is applied in below site for classification esg news issue and target company**
- [KESG Media Portal Site](http://portal.kresg.co.kr/)


# contents
1. 베이스 모델
   - 클래스별 shap 
   - 특정 클래스 문제  

2. 모델 앙상블 
   - 모델 구성 

3. 모델 심화 
   - 클러스터링 : constrative 전후 비교 
   - 



# Part1 : Optimize topic class structure by ML model 
---
 

1. 문제
- 최종적으로 뉴스 분류 업무의 자동화가 목적이다. 
- 의미론 관점에서의 분류 체계와 모델의 피쳐스페이스 관점에서의 분류 체계 
- 기존에는 사람의 기준에서 뉴스 토픽의 기준을 만들었다. 
- 하지만 피쳐 스페이스에서 정확히 분류되기 힘든 클래스들이 관측되었다. 
- 이러한 클래스를 발견하고 클래스 분류 체계의 개선을 제안



<br>

## 2. 긍정/부정 모델     
---    

긍부정 분류 모델의 경우 TF-IDF 기반의 피쳐, BERT 임베딩 벡터를 + Auto Encoder로 압축한 2000 차원의 피쳐를 Tree 계열 모델(Random Forest, LightGBM)에 실험했다. BERT기반의 성능이 더 좋았고, 결과는 아래와 같다. 

![pn_table](/assets/esg_media/topic/pn_model_confusion.png)
![pn_auc](/assets/esg_media/topic/pn_auc.png)


<br>


## 3. Base Model
---

- 먼저, Multiclass-classification 모델을 만들었다. 
- 베이스모델은 TF-IDF 기반으로 만들었다. : (document frequency ignore 기준 Threshold 0.7 ~ 0.99 를 테스트, 대부분의 주요 단어가 누락되지 않는Threhold를 찾았다.) 
- 적은 수의 클래스는 제외했다. 빨간 박스 
  
<br>

**클래스 분포**
![class_dist](/assets/esg_media/topic/클래스_분포.png)

<br>

**모델 성능**
![base_model_performance](/assets/esg_media/topic/전체모델성능.png)


<br>

### Prediction Result Pattern 

<br>

베이스 모델의 결과 데이터의 클래스는 4가지로 특성이 관측되었다. 

- 샘플 수 많음 , 정확도 높음
- 샘플 수 많음 , 정확도 낮음
- 샘플 수 적음 , 정확도 높음
- 샘플 수 적음 , 정확도 낮음



베이스 모델의 precision/recall의 관찰로 
1. 샘플수에 비례하여 precision/recall가 높지 않은 메이저 클래스가 관찰된다. -> 분석 필요 
2. 마이너 클래스 중에 특히나 precision/recall가 높은 클래스가 있다. -> 피쳐 스페이스상 Small cluster preservation이 잘 되어 있는 클래스로 보인다. 


<br>

## 4. Ensemble Model 
---

### 4.1 Class Imablance Measure
 multi-majority multi-minority 확인을 위해서 imbalnace ratio, Imbalance-degree을 살펴보았다. 
 모델의 훈련 데이터 세트를 imbalnace의 정도에 따라서 컨트롤한다. 

먼저 multi-class 이기 때문에 imbalance ratio이 아닌 imbalance degree 측정해보았다. 
imbalance degree 측정을 위해 훈련 데이터의 class 비율을 class distribution이라고 가정했다. 
그리고 distance metric으로 euclidian distance를 사용했다. 

<br>

### Data info
- k = 28 (num of class)    
- total sample size = about 100000

<br>

**Class imbalance degree**
|include 'etc' class | exclude 'etc' class|
|---|---|
|22.64|18.36|

<br>
명확한 imbalanced 문제이다. 그리고 클래스 샘플 수에 따라 multi-class classification problem $\eta_{k}$ 은 " multi-minority imbalance"로 분류 할 수 있다. 

$$\eta_{k} \text{ is multi-minority} \iff \sum^{K} \mathbb{1}\left(\eta_{i} < \frac{1}{K} \right) > \frac{K}{2}$$


### 4.2 Deal with imbalnce 

<br>

imbalance 문제를 다루기 위한 방법 중 Cost-sensitive Learning, Ensemble Methods를 사용하였다. Sampling 및 data Augmentation 기법을 사용하지 않은 이유는 다음과 같다.  

<br>

**Why not use "Sampling" and "Data Augmentation" for ?**
- Oversampling, Undersampling : majority class에서는 심한 노이즈 또는 레이블링 오차등의 퀄리티 이슈가 보이는 클래스, minority class에서는 feature space 상 cluster preservation이 잘 되어있는 것으로 보이는 클래스 들이 존재한다. 따라서 샘플링 방식이 모든 클래스에 동등한 효과를 보이기를 기대하기 어렵다. 
- Data Augmentation : 
  - ESG관련 내용에 대한 semantic meaning 연구가 없다. 
  - GPT-3나 BERT같은 언어모델로 생성하는 방법이 있으나, ESG관련 부정적 기업의 이슈는 사실에 기반한 사건이므로 생성된 데이터의 오류가 클 가능성이 있다. 추가로 시간에 따른 concept shift가 관찰되기 때문에 오버피팅의 가능성이 크다.  


---
### 4.2 Ensemble : Model split 

multi-minority



## 5. Catergory modify

- 각 클래스의 샘플의 오분류에서 편향이 발생하는지 확인을 하였다. 
- 





LightGBM + Bert Embedding을 사용했고, class weight 사용 전후를 비교한 결과는 아래와 같다. 

![](/assets/esg_media/topic/)



---


샘플 수의 많고 적음은 500개를 기준으로 하였다. 
샘플 수 많은데 정확도가 낮은 클래스를 샘플링 조사를 해보았다. 

데이터 자체에 topic이 여러개 인 경우가 많았다. 즉 사람도 정확히 맞추기 힘든 샘플들 이였다. 
이는 모델이나 데이터로 해결할 수 없는 문제이기에 클래스 분류 체계를 변경하는 것을 제안했다. 


아래는 클래스의 분류 체계 변경 후 모델 성능이다. 



뉴스 재분류 작업은 많은 리소스 투입이 필요한 일이기에 현재의 훈련 데이터셋으로 서비스 가능 한 레벨의 

모델의 평가는 f-beta (beta = 0.5)로 설정 했다. 뉴스 데이터 특성상 중복 뉴스가 나오는 경우가 많아 recall 보다는 precision이 더욱 많도록 











  