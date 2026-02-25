# AI-TrustDW (PostgreSQL)

최근 AI 모델은 무서운 속도로 발전하며 의료, 금융, 교육, 업무 자동화 등 다양한 영역에 빠르게 확산되고 있습니다.  
대규모 언어 모델(LLM)의 성능은 지속적으로 향상되고 있으며, 사용자의 의사결정 과정에 직접적인 영향을 미치는 수준에 도달했습니다.

그러나 모델의 성능 향상과는 별개로,  
사용자가 AI를 얼마나 신뢰하고 있는지, 그리고 그 신뢰가 실제 정확도와 일치하는지에 대한 문제는 여전히 중요한 주제입니다.

본 프로젝트는 **AI 모델에 대한 사용자 신뢰(Trust)와 실제 정확도(Accuracy) 간의 격차를 분석하기 위해 PostgreSQL 기반 Data Warehouse(Star Schema)를 설계하고 분석한 프로젝트**입니다.

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

프로젝트에 사용된 데이터셋 링크: https://www.kaggle.com/datasets/emanfatima2/ai-trust-insights?resource=download

![image alt](https://github.com/hyunholee9204/AI-TrustDW/blob/ee8a6c48279a47f6a4e588315c3d7121f1d87eb0/dataset1.jpg)
데이터셋의 일부 스크린샷이며 AI모델명, 사용자가 AI 답변에 부여한 신뢰 점수, 실제 답변 정확도(%) 등 23개의 컬럼이 존재합니다.

- 총 데이터 수: 1,000건
- 데이터 유형: AI 신뢰 및 회의감(Skepticism) 설문 데이터
- 주요 변수:
  - trust_score_out_of_10 (사용자가 AI 답변에 대해 부여한 신뢰 점수(0~10))
  - answer_accuracy_percentage (실제 답변 정확도(%))
  - digital_literacy_score (디지털 리터러시 등급(Low/Medium/High/Expert))
  - ai_model_name (AI 모델명)
  - performed_fact_check (사용자가 AI 답변을 추가로 검증했는지에 대한 여부)
  - 의사결정 상황 관련 변수들

---
