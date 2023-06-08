---
layout: post
title:  "ESG Media Machine-Learning Service Pipeline"
date:   2023-01-01
tags: [year22]
---

**Link**    
- [KESG Media Portal Site ](http://portal.kresg.co.kr/)       
- [Tutorial : Simple ML Pipeline with Kubernetes + Restful API ]()    


**Service Infra**    
- Kubernetes : Microk8s ( 1 Master + 2 Worker)
- Github Action 
- MicroService Architecture : Flask + Kubernetes
- Front : Dash for ProtoType
- DataBase : Postgres , mySQL
- GPU : RTX 3090 x 2    
\
**Model**    
- python, dash
- pytorch, transformer, tensorflow
- BERT (use embedding layer that fine tuned with KLUE dataset) , LightGBM, Mecab, Konlpy, Scikit-learn,


## 1. Why Kubernetes? 
- When there were many samples or intermittent network problems, people had to repair them after monitoring each time.
Therefore, self-healing-enabled Kubernetes automates these monitoring and repairs to reduce labor costs.
- Since large-scale traffic could occur in the future, the Load Balance function was required.
- Monitoring functions such as grafana can be set conveniently.
- Despite the resource limitations of single-person development, many useful functions can be easily implemented.

## 2. Why MicroService Architecture?
- Service extension is planned in the future, and it is desgined to reuse existing functions by separating them into functional units.

## 3. Why Github Action?
- Organized so that functions can be applied directly from the development server to the service server through Github Action with manifest
 
## 4. System Design 
![kubernetes_pipeline](/assets/esg_media/pipeline/kube_pipeline_trans.png)


## 4. Real Service Screen 
   
**Front**    
![front](/assets/esg_media/webpage/kresg_front.png)    
<br>
**ESG Issue Analysis**    
![issue_analysis](/assets/esg_media/webpage/kresg_issue.png)    
<br>   
**Target Company Monitoring**    
![monitoring](/assets/esg_media/webpage/kresg_monitoring.png)    
<br>    
**Target Company News List**    
![news](/assets/esg_media/webpage/kresg_news_list.png)       
<br>   
**Data Center**    
![data_center](/assets/esg_media/webpage/kresg_datacenter.png)       


