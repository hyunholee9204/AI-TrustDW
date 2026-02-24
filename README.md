# AI-TrustDW (PostgreSQL)

최근 AI 기술은 빠른 속도로 발전하며 다양한 산업 영역에 도입되고 있습니다.  
대규모 언어 모델(LLM)의 성능이 향상되면서, 사용자의 의사결정 과정에서 AI의 영향력 또한 증가하고 있습니다.

그러나 모델의 성능 향상과 사용자 신뢰 수준이 항상 일치하는 것은 아닙니다.  
AI를 실제 정확도보다 과도하게 신뢰하는 현상(Overtrust)은 중요한 사회적 리스크로 논의되고 있습니다.

본 프로젝트는 AI 모델에 대한 사용자 신뢰(Trust)와 실제 정확도(Accuracy) 간의 격차를 분석하기 위해  
PostgreSQL 기반 Data Warehouse(Star Schema)를 설계하고 분석한 프로젝트입니다.

프로젝트에 사용된 데이터셋 링크: https://www.kaggle.com/datasets/emanfatima2/ai-trust-insights?resource=download

![image alt](https://github.com/hyunholee9204/AI-TrustDW/blob/ee8a6c48279a47f6a4e588315c3d7121f1d87eb0/dataset1.jpg)

1000행의 데이터셋의 일부인 20행만 스크린샷을 찍었습니다.
AI모델명, 사용자가 AI 답변에 부여한 신뢰 점수, 실제 답변 정확도(%) 등 23개의 컬럼이 존재합니다.

---

## 1. 프로젝트 목적

이 프로젝트의 핵심 목표는 다음과 같습니다:

- 설문 기반 AI 신뢰 데이터를 정제(Cleaning)하고 구조화
- Star Schema 기반 Data Warehouse 설계
- Trust Gap 지표 생성
- 디지털 리터러시와 AI 과신(Overtrust) 관계 분석

단순 통계 분석이 아니라,  
**데이터 모델링 + ETL + 분석 스토리 도출**까지 수행한 프로젝트입니다.

---

## 2. 데이터 개요

- 총 데이터 수: 1,000건
- 데이터 유형: AI 신뢰 및 회의감(Skepticism) 설문 데이터
- 주요 변수:
  - trust_score_out_of_10
  - answer_accuracy_percentage
  - digital_literacy_score
  - ai_model_name
  - performed_fact_check
  - 의사결정 상황 관련 변수들

---
