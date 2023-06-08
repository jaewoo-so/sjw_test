---
layout: post
title:  "ESG Media Topic Classification"
date:   2023-01-01
tags: [22year_kr]
---

[KESG Media Portal Site](http://portal.kresg.co.kr/)
[Tutorial : Simple ML Pipeline with Kubernetes + Restful API ]()

Service Infra
- Kubernetes : Microk8s ( 1 Master + 2 Worker)
- Github Action 
- MicroService Architecture : Flask + Kubernetes
- Front : Dash for ProtoType
- DataBase : Postgres , mySQL
- GPU : RTX 3090 x 2 

Model 
- python, dash
- pytorch, transformer, tensorflow
- BERT (use embedding layer that fine tuned with KLUE dataset) , LightGBM, Mecab, Konlpy, Scikit-learn,


## 1. Why Kubernetes? 
- 샘플이 많은 날 또는 간헐적 네트워크 문제가 발생 시 사람이 매번 모니터링 후 수리를 해야 했다. 
따라서 Self-Healing이 가능한 쿠버네티스로 이러한 모니터링 및 수리를 자동화 하여서 인건비를 감소시킴. 
- 추후 대규모 트래픽이 발생 할 수 있어서, Load Balance 기능이 필요했다. 
- grafana 등의 모니터링 기능을 편리하게 세팅 가능.
- 1인 개발의 리소스 제한에도 불구하고 많은 유용한 기능을 쉽게 구현이 가능하다. 

## 2. Why MicroService Architecture?
- 추후 서비스 확장을 계획하고 있고, 기능 단위로 분리하여 기존의 기능을 재사용 할 수 있도록 구성했다.

## 3. Why Github Action?
- Github Action으로 개발 서버에서 도커 이미지 및 manaifest commit - push 만으로 서비스 서버에서 바로 기능이 적용 될 수 있도록 구성. 

## 4. System Design 
![kubernetes_pipeline](/assets/esg_media/pipeline/kube_pipeline_trans.png)

## 4. Real Service Screen 

**Front**
![front](/assets/esg_media/webpage/kresg_front.png)    
\

**ESG Issue Analysis**
![issue_analysis](/assets/esg_media/webpage/kresg_issue.png)    
\

**Target Company Monitoring**
![monitoring](/assets/esg_media/webpage/kresg_monitoring.png)    
\

**Target Company News List**
![news](/assets/esg_media/webpage/kresg_news_list.png)    
\ 

**Data Center**
![data_center](/assets/esg_media/webpage/kresg_datacenter.png)    

