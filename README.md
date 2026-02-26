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

### 3-1 Raw → Clean 변환

CSV 데이터를 PostgreSQL에 적재한 후, 데이터 타입 정제 및 NULL 처리를 수행했습니다.
Raw 데이터를 그대로 사용하면 타입 불일치, NULL, 문자열 불일치 문제로 인해 분석 오류가 발생할 수 있기 때문에, Clean 테이블을 별도로 구성하여 분석 안정성을 확보했습니다.

```sql
CREATE TABLE ai_trust_clean AS
SELECT
    ai_model_name,
    query_category,
    ai_confidence_percentage::NUMERIC(5,2) AS ai_confidence_percentage,
    response_character_count::INT AS response_character_count,
    has_cited_sources::BOOLEAN AS has_cited_sources,
    contains_hedging_words::BOOLEAN AS contains_hedging_words,
    includes_disclaimer::BOOLEAN AS includes_disclaimer,
    answer_detail_level,
    respondent_age_bracket,
    education_level,
    digital_literacy_score,
    ai_familiarity_level,
    decision_importance,
    urgency_level,
    belief_alignment_status,
    subject_matter_expertise,
    trust_score_out_of_10::NUMERIC(4,2) AS trust_score_out_of_10,
    performed_fact_check::BOOLEAN AS performed_fact_check,
    fact_check_method_used,
    verification_duration_mins::NUMERIC(6,2) AS verification_duration_mins,
    answer_accuracy_percentage::NUMERIC(5,2) AS answer_accuracy_percentage,
    trust_calibration_valid::BOOLEAN AS trust_calibration_valid,
    user_skepticism_category
FROM ai_trust_raw;
```

---

## 3-2 Star Schema 설계

**Dimension Tables(분석 기준 테이블) - dim_model, dim_user, dim_context**

dim_model - 어떤 AI 모델에 대한 응답? - 모델 관련 정보를 담는 테이블<br>
```sql
CREATE TABLE dim_model AS
SELECT DISTINCT
    LOWER(TRIM(ai_model_name)) AS ai_model_name,
    LOWER(TRIM(query_category)) AS query_category
FROM ai_trust_clean;

ALTER TABLE dim_model
ADD COLUMN model_id SERIAL PRIMARY KEY;
```

dim_user - 어떤 사용자 특성을 가진 사람이 평가했는지? - 사용자 특성 정보를 담는 테이블<br>
dim_context - 어떤 상황에서 AI를 사용했는지? - 의사결정 상황/심리적 맥락 정보를 담는 테이블<br>

---

## 4. Trust Gap 지표 설계
본 프로젝트에서는 단순 평균 비교를 넘어서 Trust Gap이라는 파생 지표를 생성했습니다.

```sql
trust_gap = (trust_score × 10) - answer_accuracy_percentage
```

결과값이 0보다 크면(양수) 실제 정확도보다 더 믿는다는 것을 의미(**OverTrust**)<br>
결과값이 0보다 작으면(음수) 실제보다 덜 믿는다는 것을 의미<br>
결과값이 0에 근접하면 신뢰 보정 성공을 의미<br>

---

## 4-1 AI 모델별 Trust_Gap 
어떤 모델이 가장 과신되는지 어떤 모델이 가장 저평가되는지 코드를 통해 확인해보았습니다.

```sql
SELECT
    ai_model_name,
    ROUND(AVG((trust_score_out_of_10 * 10) - answer_accuracy_percentage), 2) AS avg_trust_gap
FROM ai_trust_clean
GROUP BY ai_model_name
ORDER BY avg_trust_gap DESC;
```

![image alt](https://github.com/hyunholee9204/AI-TrustDW/blob/8ce7713bfcaab42e36b7d87b254f7b5bd60f81a4/Trustgap.png)

이 지표를 통해 3가지의 해석이 가능했습니다.
1. 모든 AI 모델의 Trust_Gap은 **0보다 크다(양수)**. <br>
> 사용자들은 전반적으로 AI를 실제 정확도보다 더 신뢰한다는 것을 의미합니다. <br>
2. Claude 모델이 가장 **과신**된다. <br>
3. ChatGPT-3.5 모델이 가장 **과신이 적다**. <br>
> ChatGPT-3.5 모델이 상대적으로 현실적인 평가를 받는 모델이라는 것을 의미합니다. <br>

---

## 5. DW 구조 기반 분석

## 5-1 모델별 평균 과신 분석
```sql
SELECT 
    m.ai_model_name,
    ROUND(AVG(f.trust_gap), 2) AS avg_trust_gap
FROM fact_ai_trust f
JOIN dim_model m ON f.model_id = m.model_id
GROUP BY m.ai_model_name
ORDER BY avg_trust_gap DESC;
```
위 코드의 수행 결과는 4-1과 동일합니다. <br>
값이 클수록 과신이 크고 값이 작을수록 과신이 낮습니다. <br>

이를 통해 Trust_Gap 지표가 가장 큰 Claude 모델은 응답 스타일이 자신감 있어 보일 가능성이 크다는 점,
GPT-4 모델은 GPT-3.5에 비해 고급 모델 이미지로 인해 신뢰가 높게 형성되어 있다는 점, ChatGPT-3.5는
이미 한계가 알려진 모델이라 기대치가 낮다는 점을 해석할 수 있었습니다.

---

## 5-2 디지털 리터러시별 과신 분석
```sql
SELECT 
    u.digital_literacy_score,
    ROUND(AVG(f.trust_gap), 2) AS avg_trust_gap
FROM fact_ai_trust f
JOIN dim_user u ON f.user_id = u.user_id
GROUP BY u.digital_literacy_score
ORDER BY avg_trust_gap DESC;
```

![image alt](https://github.com/hyunholee9204/AI-TrustDW/blob/7e04b8dc012b48116320a4fbab2021d037a7d98b/digital_literacy.png)

위 지표를 통해 **리터러시가 낮을수록 과신이 크다** 라는 결과를 도출했습니다. <br>
이는 디지털 이해도가 낮은 사람이 AI를 더 많이 믿는다는 것을 의미합니다.

이 패턴은 심리학적으로도 설명 가능합니다. <br>
**Dunning-Kruger Effect** - 지식이 부족한 사람이 자신의 이해 수준을 과대평가하는 현상

디지털 리터러시별 과신 분석 데이터는 Dunning-Kruger Effect 현상과 매우 유사하다고 볼 수 있습니다.

---

## 5-3 신뢰 vs 정확도 분해 분석
```sql
SELECT 
    u.digital_literacy_score,
    m.ai_model_name,
    ROUND(AVG(f.trust_score_out_of_10),2) AS avg_trust,
    ROUND(AVG(f.answer_accuracy_percentage),2) AS avg_accuracy
FROM fact_ai_trust f
JOIN dim_user u ON f.user_id = u.user_id
JOIN dim_model m ON f.model_id = m.model_id
GROUP BY u.digital_literacy_score, m.ai_model_name
ORDER BY u.digital_literacy_score;
```

아래 표는 디지털 리터러시 수준과 AI 모델별 평균 신뢰도 및 평균 정확도를 비교한 결과입니다.<br>
**Digital Literacy**는 디지털 정보를 이해하고, 평가하고, 비판적으로 판단할 수 있는 능력을 의미합니다.

| Digital Literacy | Model        | Avg Trust | Avg Accuracy (%) |
|------------------|-------------|-----------|------------------|
| Expert           | ChatGPT-3.5 | 7.49      | 65.93            |
| Expert           | Gemini      | 8.03      | 72.38            |
| Expert           | Mistral     | 7.48      | 68.39            |
| Expert           | Llama       | 7.66      | 72.04            |
| Expert           | GPT-4       | 8.15      | 71.55            |
| Expert           | Claude      | 8.22      | 73.54            |
| High             | Llama       | 7.64      | 67.58            |
| High             | Claude      | 7.39      | 67.94            |
| High             | ChatGPT-3.5 | 7.12      | 66.04            |
| High             | Mistral     | 7.89      | 72.93            |
| High             | Gemini      | 7.29      | 67.51            |
| High             | GPT-4       | 7.51      | 68.57            |
| Low              | Claude      | 8.38      | 70.77            |
| Low              | Mistral     | 8.62      | 73.87            |
| Low              | GPT-4       | 8.50      | 74.74            |
| Low              | Gemini      | 8.15      | 68.86            |
| Low              | Llama       | 7.89      | 64.04            |
| Low              | ChatGPT-3.5 | 8.08      | 72.21            |
| Medium           | Mistral     | 7.82      | 68.28            |
| Medium           | Gemini      | 7.75      | 68.39            |
| Medium           | Claude      | 8.26      | 69.27            |
| Medium           | ChatGPT-3.5 | 7.88      | 73.03            |
| Medium           | GPT-4       | 8.30      | 69.66            |
| Medium           | Llama       | 7.89      | 69.13            |

과신은 정확도가 낮아서 생겼을까요?<br>
아니면 신뢰 점수가 과하게 높아서 생긴걸까요?

패턴을 분석해보았습니다. <br>

1. Low 그룹(디지털 정보 해석 능력이 낮고 AI 답변을 비교적 그대로 수용할 가능성이 높은 그룹)<br>
GPT-4: Trust 8.50/accuracy 74.74
Claude: Trust 8.38/accuracy 70.77
Llama: Trust 7.89/accuracy 64.04

Low 그룹은 Trust 지표는 높으나 accuracy 지표는 64~74%

즉, 정확도가 낮아 Gap이 생긴 것이 아니라 **신뢰 점수가 전반적으로 높기 때문에 Gap이 커진 것**
Low 그룹은 AI를 전반적으로 잘 믿는 것을 확인

2. High 그룹(비교적 비판적 사고 가능, AI 응답을 그대로 믿지 않는 그룹)<br>
GPT-4: Trust 7.51/accuracy 68.57
Claude: Trust 7.39/accuracy 67.94

High 그룹은 Trust 지표는 낮아졌으나 accuracy 지표는 큰 차이가 없다.
과신 감소 원인은 **정확도 상승이 아니라 신뢰 감소**이다.
High 그룹은 더 비판적으로 본다는 것을 확인

---

## 6. 주요 인사이트

## 6-1 디지털 리터러시가 낮을수록 AI 과신(Overtrust)이 증가한다.

디지털 리터러시 수준별 평균 trust_gap 분석 결과,
Low 그룹에서 가장 높은 trust_gap이 나타났으며,
High 및 Expert 그룹으로 갈수록 trust_gap이 감소하는 패턴을 확인하였다.

이는 AI 성능 차이보다는 사용자 판단 능력 차이에 의해 과신이 발생할 수 있음을 시사한다.

---

## 6-2 과신은 모델 성능 문제보다 사용자 신뢰 구조 문제에 가깝다.

리터러시 그룹 간 평균 정확도는 큰 차이가 없었으나,
평균 신뢰 점수에서는 유의미한 차이가 나타났다.

즉, trust_gap 증가의 주요 원인은 정확도 하락이 아니라
신뢰 점수 상승에서 발생함을 확인하였다.

---

## 6-3 특정 모델은 전문가 그룹에서도 높은 신뢰를 유지한다.

Expert 그룹에서도 GPT-4, Claude 등의 모델은 평균 신뢰 점수 8점 이상을 유지하였다.

이는 모델 성능뿐 아니라 브랜드 인식 또는 이미지 효과가
신뢰 형성에 영향을 줄 가능성을 보여준다.

---

## 6-4 AI 리스크는 모델 정확도보다 사용자 신뢰 보정 실패에서 발생할 수 있다.

모델 정확도는 대체로 65~75% 범위에 분포했으나,
일부 그룹에서는 trust_gap이 크게 나타났다.

이는 AI 리스크가 단순 성능 문제보다
신뢰-정확도 불일치(Trust Calibration Failure)에서 비롯될 수 있음을 의미한다.
