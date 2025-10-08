# | Passage Retrieval – 의미 기반 문서 검색 (Dense Embedding)

## 🧾 핵심 정리 | Sparse vs Dense

| 항목    | Sparse Embedding | Dense Embedding    |
| ----- | ---------------- | ------------------ |
| 예시    | TF-IDF, BM25     | BERT, DPR, ColBERT |
| 표현 기반 | 단어 빈도 기반         | 의미 기반 벡터 공간        |
| 학습    | 비학습 (통계)         | 학습 기반 (신경망)        |
| 확장성   | 새로운 단어 추가 시 불가   | 추가 학습으로 보완 가능      |
| 적합 문제 | 정확한 키워드 매칭       | 의미 유사 검색, 질의응답     |

---

> **핵심 주제:** Sparse 방식의 한계를 극복하기 위해 단어 의미 공간을 학습해 유사도를 계산하는 Dense Embedding 기반 문서 검색

---

## 1️⃣ Introduction to Dense Embedding

**조밀한 임베딩 (Dense Embedding)**

### 💡 Dense Embedding이란?

* Sparse representation (예: TF-IDF) 과 보완적인 개념
* 작은 차원의 고밀도 벡터 (length ≈ 50 ~ 1000)
* 각 차원이 특정 단어 (term)에 직접 대응하지 않음
* 대부분의 요소가 **non-zero 값**을 가짐
* 학습을 통해 **의미적 연관성**을 반영

| 구분 | Sparse Embedding  | Dense Embedding  |
| -- | ----------------- | ---------------- |
| 차원 | 매우 높음 (수만 차원)     | 낮음 (수백 차원)       |
| 요소 | 대부분 0             | 대부분 non-zero     |
| 특징 | 단어 단위 일치 기반       | 의미 기반 연관성 반영     |
| 학습 | 고정된 벡터            | 추가 학습 가능         |
| 장점 | 명확한 term 매칭 성능 우수 | 유사 단어 간 관계 이해 우수 |

> 📘 요약: Dense Embedding은 “비슷한 의미의 단어 또는 문장”을 벡터 공간에서 가까이 위치시키는 것이 핵심이다.

---

## 2️⃣ Training Dense Encoder

**Dense Encoder 모델 학습 및 훈련 방법**

### 🧠 Dense Encoder란?

* 문장이나 문서를 Dense Embedding 벡터로 변환하는 모델
* 주로 **BERT**, RoBERTa 등 PLM (Pre-trained Language Model) 사용
* `[CLS]` 토큰의 출력 벡터를 대표 값으로 활용

```text
입력: [CLS] Query [SEP] Passage [SEP]
출력: CLS 토큰 임베딩 (문서/질의 표현)
```

---

### 🎯 학습 목표

**연관된 question – passage 쌍은 가깝게**,
**연관되지 않은 쌍은 멀게** 학습시킴.

* 목적 함수(Objective): Positive passage에 대한 Negative Log Likelihood (NLL) loss
* 거리 척도: 코사인 유사도 또는 내적 (inner product)

---

### ⚙️ Negative Sampling

> 모델이 혼동하지 않도록 유사하지만 정답이 아닌 문장을 샘플링

* **Random Negatives:** Corpus 내 임의의 문서 추출
* **Hard Negatives:** TF-IDF 등으로 유사도가 높지만 정답이 아닌 문서 선정

---

### 📈 평가 지표

**Top-k Retrieval Accuracy**

> 검색된 passage 상위 k개 중 실제 답을 포함한 passage의 비율

예:
Top-5 accuracy = 5개 중 정답 passage 1개 이상 포함 시 1 / 총 쿼리 수

---

## 3️⃣ Passage Retrieval with Dense Encoder

**Dense Embedding 기반 문서 검색 흐름**

### 🔄 Inference Flow

1. Passage 전체를 Dense Encoder로 임베딩하여 벡터 저장
2. 사용자의 Query 역시 임베딩
3. Query 벡터와 가장 가까운 (코사인 유사도 높은) Passage 탐색
4. 선택된 Passage를 MRC 모델에 입력하여 정답 생성

---

### 🔍 From Retrieval to Open-Domain QA

* Dense Retriever로 문서를 선택
* 선택된 문서를 기반으로 **MRC 모델**이 세부 답변 도출
* 즉, “검색 → 이해 → 응답”의 2-단계 Pipeline 구조

---

### 🚀 Dense Encoding 성능 향상 전략

1. **학습 전략 개선** — e.g. DPR (Dense Passage Retriever)
2. **모델 개선** — 더 큰 PLM (RoBERTa-large, E5 등)
3. **데이터 개선** — 다양한 domain 추가 및 전처리 보강

---

> ✅ Dense Embedding의 핵심은 단어의 “표면적 일치” 가 아닌
> “의미적 유사성” 을 기반으로 검색을 수행한다는 것이다.
> 이 개념은 이후의 Open-Domain QA, Retrieval-Augmented Generation (RAG) 등으로 확장된다.
