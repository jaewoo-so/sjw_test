---
layout: post
title:  "A tissue origin prediction device, method of predicting the tissue origin using a genome data, and computer program"
tags: [publication_patents]
---
1020200076756 · Filed Jun 23, 2020

[Front]('./assets/../../../assets/publication_patents/patent_pdl1/patent_pdl1_front.png')

# Summary 
[2018.05-2018.07] TCGA,GEO데이터를 이용해 33종 암의 primary site 예측 및 similarity mapping 모델 연구

- 모델 성능 측정 방법 디자인을 위한 데이터 shift 분석
- gene expression을 이용한 예측 모델 개발(deep learning모델, GBDT모델, SVM, 선형모델 등 비교)
- multi-minority 문제 해결을 위한 gene expression, DNA methylation, microRNA expression 의 비교.
- cell type별 및 다른 시퀀싱 파이프라인 데이터를 사용해 교차검증 진행.


### Data
- RNA, mRNA, methylation
- 전이암 환자들의 원발 부위 정보

# Summary Idea & Solution
- 피쳐 Selection이 핵심. 모든 피쳐가 같은 타입,특징이다. 
- 피쳐 수 80만개 이상, 샘플 수를 적절하게 줄여야 한다. 
- 다음의 기준으로 샘플 수를 줄여 나갈 수 있다. 아래 방법은 모두 효과가 있는 것을 확인했다. 
  - sparsity
  - standard_diviation
  - entorpy / gini ( tree 기반의 모델에서 얻을 수 있다. ) : consistancy 보장이 안된다는 단점이 있다.
  - lime or shap : 연산시간의 오래걸리는 단점이 있다. 

- 대부분의 피쳐가 sparse하다. 
- 실제로 모델로 tissue origin을 찾아야 한다. 
- 

# Detail 

샘플 수 대비 성능 등의 그래프 를 올리면 좋을듯. 
