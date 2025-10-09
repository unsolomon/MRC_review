# 07. Linking MRC and Retrieval (ODQA)

> 오픈 도메인 질문응답(Open-domain QA)과 기계독해(MRC), 문서 검색 시스템의 연결

## 📘 전체 요약

| 구분 | 핵심 내용 |
|------|------------|
| **ODQA** | 외부 지식 기반의 질의응답 시스템 |
| **Retriever** | 관련 문서 검색 (TF-IDF, BM25, Dense Encoder 등) |
| **Reader** | 검색된 문서 내 정답 추론 (BERT, RoBERTa 등) |
| **Distant Supervision** | 자동 데이터셋 생성 기법 |
| **Inference** | 검색 → 독해 → 최종 답변 추론 과정 |
| **Recent Trend** | Dense 기반 ODQA, Multi-passage Reader, Joint Learning 구조 |

---

---

## 1️⃣ Introduction to ODQA (Open-domain Question Answering)

### 🧠 개념 정리

- **MRC (Machine Reading Comprehension)**  
  주어진 지문 내에서 질문의 답을 찾는 과정.

- **ODQA (Open-domain Question Answering)**  
  사전에 지문이 주어지지 않으며, 방대한 외부 지식(예: Wikipedia)을 기반으로 답을 찾아내는 시스템.  
  현대의 검색엔진(Google, Naver 등)은 이 방식을 활용하여 질문에 대한 직접적인 답변을 제공.

---

## 2️⃣ Historical Background

### 📜 TREC QA Tracks (1999–2007)
정보 검색(IR) 중심의 QA 시스템에서 짧은 형태의 답변을 제공하는 구조로 발전함.

**구성요소**
1. **질문 처리 (Question Processing)** – Query 생성, 답변 유형 분류  
2. **패시지 검색 (Passage Retrieval)** – 문서 내 관련 문장 탐색  
3. **답변 처리 (Answer Processing)** – 문서 내에서 규칙 기반으로 답 추출  

📌 대표 사례:  
- **IBM Watson (2011, Jeopardy!)**  
  “DeepQA Project”로 사람보다 뛰어난 성능을 보인 초기 ODQA 시스템.

---

## 3️⃣ Retriever–Reader Approach

| 구성 요소 | 역할 |
|------------|------|
| **Retriever** | 대규모 문서 집합에서 질문과 관련 있는 문서를 검색 |
| **Reader** | Retriever가 가져온 문서에서 질문의 정답을 추론 |

### ⚙️ 학습 방식

- **Retriever**
  - Sparse 방식: TF-IDF, BM25 (학습 불필요)
  - Dense 방식: BERT, DPR 등 Encoder 학습 필요  
- **Reader**
  - SQuAD, KorQuAD 등의 MRC 데이터셋으로 Fine-tuning 수행

---

## 4️⃣ Distant Supervision

> 질문–답변 쌍 데이터로부터 자동으로 MRC 학습 데이터를 생성하는 방식

**절차 요약**
1. 질문을 기반으로 위키피디아 등에서 관련 문서 후보 검색  
2. 질문과 무관하거나 부적합한 문서 제거  
3. 질문과 가장 높은 관련성을 가지는 단락을 **Supporting Evidence(지지 근거)** 로 선택  

💡 장점:  
별도의 수작업 라벨링 없이 ODQA용 학습 데이터 자동 구축 가능.

---

## 5️⃣ Inference (추론 단계)

1. Retriever가 질문과 관련된 **Top-k 문서**를 검색  
2. Reader가 각 문서를 분석하여 답변 후보 생성  
3. 가장 높은 스코어를 얻은 답변을 최종 답변으로 선정  

> Retriever는 “어디서 답을 찾을지”,  
> Reader는 “어떤 내용이 답인지”를 결정하는 협업 구조.

---

## 6️⃣ Issues & Recent Approaches

### 📍 Passage Granularity
- 문서 단위를 **Document / Paragraph / Sentence** 중 어디로 설정할지 결정
- 단위가 세밀할수록 연산량은 증가하지만 정확도 향상 가능

| 단위 | 데이터 규모 예시 | 특징 |
|------|------------------|------|
| Article | 약 5M | 전체 문서 수준, 빠르지만 세밀도 낮음 |
| Paragraph | 약 30M | 문단 단위의 검색, 일반적으로 가장 활용 많음 |
| Sentence | 약 75M | 문맥 연결성 약하나 정밀한 검색 가능 |

---

### 🧩 Single vs Multi-passage Training

| 방식 | 설명 | 장단점 |
|------|------|--------|
| **Single-passage** | 개별 패시지별로 독립적인 예측 수행 | 단순하지만 문맥 손실 |
| **Multi-passage** | 여러 패시지를 묶어 Reader가 통합 분석 | 풍부한 문맥 반영, 연산량 많음 |

---

### ⚖️ Passage Importance
Retriever가 제공한 **유사도 점수(Retrieval Score)** 를 Reader의 입력에 포함해,  
패시지의 중요도를 가중치로 반영하는 방법.

---

## 7️⃣ Recent Research Directions

- **End-to-End ODQA** : 검색과 독해를 하나의 모델에서 수행 (예: BERTserini, DPR, RAG)
- **Retriever–Reader Joint Learning** : 검색과 독해 모듈을 동시에 학습
- **Dense Retrieval + Generation** : Dense 기반 검색 결과를 활용해 생성형 답변 생성 (RAG, FiD 등)

---

## 📎 참고 자료

- Chen et al. (2017). *Reading Wikipedia to Answer Open-domain Questions*  
- Yang et al. (2019). *End-to-End Open-Domain Question Answering with BERTserini*  
- Zeng et al. (2020). *A Survey on Machine Reading Comprehension*  
- [FAISS GitHub](https://github.com/facebookresearch/faiss)  
- [ODQA Tutorial (ACL 2020)](https://github.com/danqi/acl2020-openqa-tutorial)

---

### ✍️ Study Notes

- ODQA는 **검색 시스템(IR)** 과 **독해 시스템(MRC)** 의 융합이다.  
- Dense Retriever를 기반으로 문서 의미를 벡터 공간에 매핑 → Reader가 의미 기반 추론 수행.  
- “Retriever가 ‘어디서 찾을지’를, Reader가 ‘무엇이 답인지’를 결정한다.”  
- 최근엔 RAG, FiD, Atlas처럼 **Retrieval-Augmented Generation** 방식으로 발전 중.

