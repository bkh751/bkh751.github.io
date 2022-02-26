---
layout: post
title:  "featurestore"
date:   2022-02-26 01:18:45 +0900
categories: mlops
---

먼저 피처 스토어가 무엇이고, 왜 필요한지, 그리고 언제 도입을 해야하는지에 대해 설명이 간략히 필요해보여서 featurestore.org 의 글을 몇가지 정리해보았습니다.

원문 https://www.featurestore.org/what-is-a-feature-store

## ML에서 피처란 무엇인가?

ML 혹은 MLOps 에서, 예측 모델을 사용하려고하면 모델에 피처 값을 주입해야합니다.

단순히 말해서 피처는 데이터의 특성입니다. 예를들면 필터가 적용된 이미지, 알고리즘을 통해 추출된 숫자 정보들이 피처입니다. 이러한 정보들은 사진의 픽셀값 혹은 엑셀시트 로우값으로 저장해서 씁니다.

ML 모델은 피처가 많이 필요합니다.

피처를 만들기 위한 데이타는 데이타 소스로부터 가져오고 피처에 대한 연산을 한뒤 저장이 되는 과정이 있어서 파일 구조를 잘 정리해서 관리해야합니다. (이 과정을 피처 엔지니어링이라고 합니다.) 그래서 ML 에서는 모델 학습을 위해서는 파일구조를 잘만들어둬야하는 부담이 있습니다.

## 피처 스토어는 무엇인가?

피처스토어는 피처들을 저장하고 데이타 사이언티스트가 학습을 하기위한 피처를 사용하고, 학습된 모델을 가진 어플리케이션이 예측을 하기 위한 피처를  주입 받기 위해 사용이됩니다. 여러 데이타 소스로부터 받은 피처 그룹들을 생성하고, 업데이트가 가능하며 새로운 데이터셋을 생성해둔 피처그룹들을 이용해서 생성/업데이트 가능합니다. 혹은  애플리케이션에서 예측 모델을 돌리기 위해서 피처를 획득하기 위해서도 사용합니다.

최초의 피처 스토어는 우버가 2017에 발표한 Michelangelo Palette 입니다. 문서화, 디스커버리, 그리고 피처 재사용, 배치 애플리케이션 혹은 온라인 애플리케이션으로에서 사용되는 피처 검증을 위한 목적으로 개발 되었습니다. 또한 높은 throughput 과 low latency 를 기반으로한 batch API 를 point-in-time 학습에 사용하거나 batch 예측을 위한 피처 주입을 할 수 있습니다. 

## 예를들면, 언제 피처 스토어를 쓰면 좋은가?

사용자별로 개인화된 검색 쿼리를 날릴때를 예를 들어 설명해보겠습니다. 사용자가 검색쿼리를 한번 할때, 보통 마이크로서비스에서의 stateless web app 에서는 DB, KV 스토어 혹은 메시지버스로부터 결과 값을 가져오게됩니다. 이러한 검색은  text 와 userID/sessionID 정도의 정보만 가지고있어서 정보량이 적습니다.(**information-poor signal 이라고합니다)**
 . 그러나 useID 와 sessionID 와 같은 **information-poor signal은 피처 스토어를 통해 피처 정보들을 함께 가지고있는 information-rich signal로 변환이 가능합니다.** 피처의 내용은 예를들어 user 의 주문 이력(history)나 어떤 물건이 가장 잘 팔리는지(context) 알려줍니다. 애플리케이션은 이러한 history 와 context 를 기반을 가진 피처를 받아 AI-enabled 가 됩니다.

## 피처 스토어는 어떤 특징이 있나?

원문 [https://www.hopsworks.ai/post/how-to-build-your-own-feature-store](https://www.hopsworks.ai/post/how-to-build-your-own-feature-store)

- 일관된 피처 엔지니어링,
- 피처 재사용
- 피처 서빙
- 피처 스토어로부터 미리 연산된 피처를 받아 좀 더 쉬운 EDA(탐색적 데이타 분석)가 가능
- Point-in-Time Correct Training Data (no data leakage)
- 보안과 거버넌스
- 학습데이터셋 재생산


## 그러면 피처스토어를 도입은 무엇을 해결하는가?

피처스토어는 여러가지 패키지로 구성됩니다. 스토리지 역할을 하는 패키지는 온라인 DB, 혹은 DB 대신에 FeatureForm 을 가진 가상 피처스토어가 있는데, Rasgo 와 Snowflake 와 같은 Data Warehouse 와 같이 통합되어있는 형태입니다. 연산을 할 수있는 패키지는 여러 데이타소스(Data warehouses, data lakes, event buses, etc) 로 연산 작업을 돌릴 수 있는 스파크가 있습니다.

산업에서 쓰이는 데이타 인프라와 머신러닝 툴에서 피처 스토어는 플러그인 형태로 제공이됩니다. 

일반적으로 이러한 인프라에서는 operational data 와 analytical data 가 있습니다. operational data 는 business 로직에 관여되고, analytical data 는 data warehouse or datalake 에 있는 데이타로 분석과 business 로직 최적화를 위해 사용됩니다. ETL 을 수행하는 Data Pipelines 는 operational data 스토어로부터 데이타를 가져와, 변환과정을 거치게 되면 data warehouse or datalake 로 write 합니다. 일련의 과정을 Feature Pipeline 이라고합니다. 그러나 이러한 Feature Pipeline 은 무분별하게 피처별로 생성이 될 가능성이 있고 데이타 과학자가 어떤 피처가 사용가능한지, 그리고 같은 피처를 재사용할 수 있는 일관된 방법을 찾기가 어렵습니다. 그래서 이러한 과정을 해결하기 위해  Operational Data 와 Analytical Data 를 구분해서 모아 둘 수 있는 피처스토어를 도입함으로써 해결이 가능합니다. 

또한 데이타 엔지니어와 데이타 사이언티스트간의 코딩 언어를 똑같이 쓰지 않아도 되게끔 해줍니다. 데이타 엔지니어는 analytic data 에 대한 책임이 있고, 데이타 사이언티스트의 도움과 함께 피처스토어로 옮기는 일을 합니다. 

데이타 사이언티스트는 추가적인 피처 파이프라인을 만드고, 피처를 재작성하고, 모델학습을 위해 피처를 사용합니다. 그러면 ML 엔지니어들이 이러한 모델을 프로덕션에 사용합니다. DE, DS, ML엔지니어가 각각 다른 언어를 쓰면서 피처스토어를 모두가 사용하는 하나의 플랫폼으로 사용해서 협업이 쉬워집니다. 

## 어떤 Feature Store 를 쓰는게 좋을까 (TBD)

Sage Maker Feature Store https://aws.amazon.com/ko/sagemaker/feature-store/
Feast https://docs.google.com/presentation/d/1Y4DghcVfGCfyDfwRZ_UHpeShuLrwh1gJxlWjyF8K3pI/edit#slide=id.ga455c5eaff_2_69

[참조링크]

[https://www.hopsworks.ai/post/feature-store-the-missing-data-layer-in-ml-pipelines](https://www.hopsworks.ai/post/feature-store-the-missing-data-layer-in-ml-pipelines)

[https://medium.com/data-for-ai/what-is-a-feature-store-for-ml-29b62580af5d](https://medium.com/data-for-ai/what-is-a-feature-store-for-ml-29b62580af5d)

 [https://medium.com/data-for-ai/feature-store-101-b964373891c4](https://medium.com/data-for-ai/feature-store-101-b964373891c4)
