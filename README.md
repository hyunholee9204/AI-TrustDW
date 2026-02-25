# AI-TrustDW (PostgreSQL)

최근 AI 모델은 무서운 속도로 발전하며 의료, 금융, 교육, 업무 자동화 등 다양한 영역에 빠르게 확산되고 있습니다.  
대규모 언어 모델(LLM)의 성능은 지속적으로 향상되고 있으며, 사용자의 의사결정 과정에 직접적인 영향을 미치는 수준에 도달했습니다.

그러나 모델의 성능 향상과는 별개로,  
사용자가 AI를 얼마나 신뢰하고 있는지, 그리고 그 신뢰가 실제 정확도와 일치하는지에 대한 문제는 여전히 중요한 주제입니다.

본 프로젝트는 **AI 모델에 대한 사용자 신뢰(Trust)와 실제 정확도(Accuracy) 간의 격차를 분석하기 위해 PostgreSQL 기반 Data Warehouse(Star Schema)를 설계하고 분석한 프로젝트**입니다.

---

## 1. 프로젝트 목표

- 설문 기반 AI 신뢰 데이터를 정제(Cleaning)
- Star Schema 기반 Data Warehouse 설계
- Trust Gap 파생 지표 생성
- 디지털 리터러시와 AI 과신(Overtrust) 관계 분석
- 단순 통계 분석을 넘어 데이터 모델링 + ETL + 인사이트 도출 수행

---

## 2. 데이터 개요

프로젝트에 사용된 데이터셋 링크: https://www.kaggle.com/datasets/emanfatima2/ai-trust-insights?resource=download

![image alt](https://github.com/hyunholee9204/AI-TrustDW/blob/ee8a6c48279a47f6a4e588315c3d7121f1d87eb0/dataset1.jpg)
데이터셋의 일부 스크린샷이며 AI모델명, 사용자가 AI 답변에 부여한 신뢰 점수, 실제 답변 정확도(%) 등 23개의 컬럼이 존재합니다.

- 총 데이터 수: 1,000건
- 데이터 유형: AI 신뢰 및 회의감(Skepticism) 설문 데이터
- 주요 변수:

| 컬럼 | 설명 |
|------|------|
| ai_model_name | 사용된 AI 모델 |
| trust_score_out_of_10 | 사용자가 부여한 신뢰 점수 |
| answer_accuracy_percentage | 실제 정답 기준 정확도 |
| digital_literacy_score | 디지털 리터러시 수준 |
| performed_fact_check | 추가 검증 여부 |
| decision_importance | 의사결정 중요도 |
| user_skepticism_category | 사용자의 기본 회의 성향 |

이 데이터는 AI 응답 특성, 사용자 특성, 의사결정 상황, 실제 정확도를 동시에 포함하는 다차원 분석 구조를 가지고 있습니다.

---

## 3. 데이터 엔지니어링 과정

## 3.1 Raw → Clean 변환

CSV 데이터를 PostgreSQL에 적재한 후, 데이터 타입 정제 및 NULL 처리를 수행했습니다.

```sql
CREATE TABLE ai_trust_clean AS
SELECT
    ai_model_name,
    query_category,
    ai_confidence_percentage::NUMERIC(5,2),
    response_character_count::INT,
    trust_score_out_of_10::NUMERIC(4,2),
    answer_accuracy_percentage::NUMERIC(5,2),
    performed_fact_check::BOOLEAN
FROM ai_trust_raw;
