# 📚 Passage Retrieval – Sparse Embedding

> **Lecture 4. Passage Retrieval: Sparse Embedding**
> Instructor: Minjoon Seo, KAIST AI 대학원
> ⓒ NAVER Connect Foundation

---
## ✅ 정리 요약

| 구분                    | 핵심 내용                           |
| --------------------- | ------------------------------- |
| **Passage Retrieval** | Query와 가장 유사한 문서를 찾는 과정         |
| **Sparse Embedding**  | 단어 등장 여부 기반 벡터 표현 (BoW, TF-IDF) |
| **TF-IDF**            | 단어 중요도 = 빈도(TF) × 희귀도(IDF)      |
| **장점**                | 구현 단순, 효율적, 고해상도 단어 비교          |
| **단점**                | 단어 의미 반영 불가 (동의어, 문맥 무시)        |
| **BM25**              | TF-IDF 개선 버전 (문서 길이·포화 보정)      |

---

## 1️⃣ Introduction to Passage Retrieval

### 🧠 개념 정의

**Passage Retrieval**

> 주어진 **질문(Query)** 에 대해 **가장 관련 있는 문서(Passage)** 를 찾아내는 과정.

이는 오픈 도메인 질의응답(ODQA, *Open-Domain Question Answering*)의 첫 단계로,
수많은 문서 중 **질문과 연관된 후보 문서**를 빠르고 정확히 검색해야 한다.

---

### 🧩 Passage Retrieval with MRC

MRC(Task)와 결합하면 아래와 같은 2단계 구조로 구성된다.

1. **Stage 1 – Passage Retrieval:**

   * 대규모 문서 집합에서 질문에 맞는 문서 후보를 찾음
2. **Stage 2 – Machine Reading Comprehension:**

   * Retrieval로 찾은 문서 내에서 **정답을 추출 또는 생성**

> 💡 *즉,*
> Passage Retrieval은 MRC의 “입력 정제 단계”로,
> 모델이 답변할 문맥(Context)을 효율적으로 좁혀주는 역할을 한다.

---

### 🔍 Retrieval 과정 개요

1. Query와 각 Passage를 **벡터(embedding)** 로 변환
2. 두 임베딩 간 **유사도(similarity)** 계산
3. 가장 유사도가 높은 문서를 선택 (Top-k ranking)

> 예시:
> `질문: "BTS의 리더는 누구인가?"`
> → 문서 임베딩 중 “BTS”, “RM”, “리더”와 연관 높은 Passage 선택

---

## 2️⃣ Passage Embedding & Sparse Embedding

Passage Retrieval의 핵심은 **문서 임베딩(Embedding)** 방법이다.

---

### 📘 Passage Embedding

> 텍스트(문서, 문장)를 벡터로 표현하는 방법.

이 벡터 공간에서 **Query–Passage 간의 거리(distance)** 혹은 **유사도(similarity)** 를 계산하여
검색 대상 문서의 관련도를 평가한다.

**주요 방법**

* Sparse Embedding (희소 벡터 표현)
* Dense Embedding (밀집 벡터 표현, e.g. DPR, ColBERT 등)

이번 강의에서는 **Sparse Embedding**에 초점을 맞춘다.

---

### 🌿 Sparse Embedding 소개

> 문서 내 단어를 **0과 1**, 혹은 **빈도수 기반 값**으로 표현하는 고차원 벡터 방식.

즉, 각 단어가 벡터 공간의 차원을 구성한다.
문서가 포함하는 단어에 따라 **단어 존재 여부(BoW, Bag of Words)** 또는 **빈도(TF)** 로 값을 채운다.

---

#### 🧮 BoW (Bag of Words) 방식

문서를 단어의 집합으로 단순화하는 모델.
단어의 순서 정보는 무시되며, **단어의 등장 여부/빈도만 고려.**

* **Unigram (1-gram):**
  `It was the best of times` → `It`, `was`, `the`, `best`, `of`, `times`
* **Bigram (2-gram):**
  `It was the best of times` → `It was`, `was the`, `the best`, `best of`, `of times`

> ✅ n-gram 수가 커질수록 특징 차원이 증가하며, 보다 정교한 문맥 표현이 가능하지만
> 연산량과 메모리 사용량도 함께 증가한다.

---

#### ⚙️ Sparse Embedding 특징 요약

| 항목                 | 설명                                           |
| ------------------ | -------------------------------------------- |
| **Embedding 차원 수** | 전체 용어 수와 동일 (terms = dimensions)             |
| **n-gram 증가**      | 차원 수 급격히 증가                                  |
| **장점**             | 특정 단어 overlap을 정확히 파악 가능                     |
| **단점**             | 유사 의미 단어 간(예: “buy” vs “purchase”) 의미적 비교 불가 |

> 💡 *요약:* Sparse Embedding은 “단어의 존재 여부”를 직접적으로 표현하지만,
> **의미적 유사성(semantic similarity)** 은 반영하지 못한다.

---

## 3️⃣ TF-IDF (Term Frequency – Inverse Document Frequency)

Sparse Embedding의 대표적 수치화 방법 중 하나가 **TF-IDF** 이다.

---

### ⚙️ 정의

TF-IDF는 문서 내 특정 단어의 **중요도(weight)** 를 수치로 표현한다.

> **TF (Term Frequency)**: 단어가 문서에 얼마나 자주 등장하는가
> **IDF (Inverse Document Frequency)**: 단어가 전체 문서에서 얼마나 희귀한가

두 값을 곱하여 특정 문서 내 단어의 “정보량”을 측정한다.

---

### 🧩 Term Frequency (TF)

> 단어가 특정 문서 내 등장한 빈도

* **Raw Count:** 등장 횟수
* **Length-normalized:** 문서 길이로 정규화

[
\mathrm{TF}(t, d) = \frac{\text{단어 t의 등장 횟수}}{\text{문서 d의 전체 단어 수}}
]

> 다른 변형 방식: binary(존재 여부), log-scaling 등

---

### 🔍 Inverse Document Frequency (IDF)

> 단어의 정보량을 나타내며, **모든 문서에 흔하게 등장할수록 가치는 낮다.**

* **N:** 전체 문서 수
* **DF(t):** 단어 *t*가 등장한 문서 수

[
\mathrm{IDF}(t) = \log \left( \frac{N}{1 + \mathrm{DF}(t)} \right)
]

> “the”, “is”, “and”처럼 대부분 문서에 존재하는 단어는 IDF가 0에 가까움
> 반면 “BTS”, “Heisenberg”처럼 희귀 단어는 높은 IDF 값을 가짐.

---

### 💡 TF-IDF 결합

[
\mathrm{TF\text{-}IDF}(t, d) = \mathrm{TF}(t, d) \times \mathrm{IDF}(t)
]

| 단어    | TF | IDF | TF-IDF | 해석     |
| ----- | -- | --- | ------ | ------ |
| `the` | 높음 | 낮음  | 낮음     | 정보량 낮음 |
| `BTS` | 낮음 | 높음  | 높음     | 중요 단어  |

---

### 📘 TF-IDF 계산 예시

| 문서 제목 | 문서 내용                     |
| ----- | ------------------------- |
| 음식    | 주연은 과제를 좋아한다              |
| 운동    | 주연은 농구와 축구를 좋아한다          |
| 영화    | 주연은 어벤져스를 가장 좋아한다         |
| 음악    | 주연은 BTS의 뷔가 가장 잘생겼다고 생각한다 |

1. 각 문서에서 단어 등장 빈도 계산 (TF)
2. 전체 문서 내 등장 문서 수 기반 IDF 계산
3. 두 값 곱해 문서별 **TF-IDF 벡터** 생성
4. Query의 TF-IDF와 문서의 TF-IDF 벡터 간 **유사도(cosine similarity)** 계산

---

### 🧠 TF-IDF 기반 문서 검색 단계

1. 사용자가 입력한 질의를 토큰화
2. 기존 사전에 없는 토큰 제거
3. 질의를 하나의 “문서”로 간주하고 TF-IDF 계산
4. 질의 TF-IDF와 각 문서의 TF-IDF 벡터 내적 계산
5. 유사도 점수가 가장 높은 문서를 결과로 반환

> 예시
> **Query:** “주연은 BTS의 누구를 가장 잘생겼다고 생각하나?”
> → 가장 높은 TF-IDF 유사도를 가진 문서: “음악”

---

### 📊 TF-IDF의 한계와 개선 (BM25)

**BM25**는 TF-IDF의 확장된 형태로,
문서 길이와 단어의 포화 빈도(saturation)를 함께 고려한다.

| 특징           | 설명                     |
| ------------ | ---------------------- |
| **문서 길이 보정** | 짧은 문서에서의 등장 단어에 가중치 부여 |
| **TF 한계 설정** | 너무 자주 등장하는 단어에 포화값 적용  |
| **현업 적용 예시** | 검색엔진, 추천 시스템, 논문 검색 등  |

> 📌 *Tip:* Pyserini 라이브러리의 BM25 구현으로 실험 가능
> [Pyserini BM25 Example (MS MARCO)](https://github.com/castorini/pyserini/blob/master/docs/experiments-msmarco-passage.md)

---


## 🚀 실무 및 프로젝트 활용 포인트

1. **TF-IDF + Cosine Similarity**로 간단한 문서 검색기 구현 가능
2. **BM25**는 현재도 검색엔진 기본 알고리즘으로 사용
3. Sparse Embedding은 **Dense Retrieval (BERT-based)** 모델과 비교 실험 시 baseline으로 유용
4. KorQuAD, MS MARCO 등 QA 데이터셋에서 **retriever 모델 사전 단계로 활용** 가능

---

📚 **참고문헌 및 출처**

* Manning et al., *Introduction to Information Retrieval* (Cambridge University Press, 2008)
* Kim et al., *Research paper classification systems based on TF-IDF and LDA schemes*, *HCIS Journal*, 2019
* [Pyserini GitHub Repository](https://github.com/castorini/pyserini)
* [Wikipedia: TF–IDF](https://en.wikipedia.org/wiki/Tf–idf)
