# 📘 10강. QA with Phrase Retrieval

## 🔹 주제
전체 문서가 아닌 **구문(Phrase)** 단위로 검색하여 질문에 답하는 **오픈 도메인 질문답변(ODQA)** 시스템 설계

## 🧩 핵심 요약

| 구분 | 주요 내용 | 프로젝트 적용 포인트 |
|------|------------|----------------------|
| **Phrase Retrieval** | 문서 단위가 아닌 Phrase 단위로 검색 | 데이터 단위 분할/검색 최적화 설계 |
| **Dense + Sparse 결합** | 의미적 + 어휘적 정보 결합 | 의미 기반 질의응답 시스템 구축 |
| **FAISS 기반 검색** | 대규모 데이터 효율적 검색 | 백엔드 검색 엔진 구현 시 활용 |
| **DensePhrases/ColBERT** | 최신 효율적 검색 모델 | 모델 경량화 및 실시간 응답 설계 |
| **Decomposability Gap 해결** | Question-Answer 간 attention 복원 | 인코딩 구조 설계 참고 |

---

## 🧩 1. Phrase Retrieval in Open-Domain Question Answering

### 🔸 기존 Retriever-Reader 방식의 한계
1. **Error Propagation**
   - 상위 5~10개 문서만 Reader에 전달되어 **초기 검색 오류가 전파됨**  
2. **Query-dependent Encoding**
   - 질문에 따라 **답변 스팬의 인코딩이 달라져** 일관성 있는 검색 어려움

### 🔸 해결책: Phrase Retrieval
- 문서를 **문구(Phrase)** 단위로 분해하고, 각 phrase를 독립적으로 벡터화  
- 질문도 벡터로 변환하여 **Nearest Neighbor Search (NNS)** 수행 → 가장 유사한 구문 반환  
- **Query-Agnostic Decomposition**
  - 질문과 무관하게 **모든 문구를 미리 인덱싱**
  - 질문이 들어오면 벡터 공간에서 직접 검색  

> ✅ **핵심 아이디어:**  
> “Retrieve → Read” 두 단계를 거치지 않고, **‘정답 구문’을 바로 검색**

---

## 🧠 2. Dense-sparse Representation for Phrases

### 🔸 Dense vs Sparse 표현
| 구분 | Dense 표현 | Sparse 표현 |
|------|-------------|-------------|
| 특징 | 의미적/통사적 정보 표현 | 어휘적(lexical) 정보 표현 |
| 예시 | BERT, SpanBERT | TF-IDF, BM25 |
| 장점 | 문맥 이해에 강함 | 단어 기반 정합에 강함 |

> 두 방식을 **결합(Dense + Sparse)** 하면 문맥과 단어 정합을 모두 고려 가능

---

### 🔸 Phrase & Question Embedding
- Phrase와 Question을 각각 임베딩  
- Dense 표현은 **BERT 기반**, Sparse 표현은 **n-gram 가중치 기반**

### 🔸 Dense Representation (DenSPI, 2019)
- BERT 기반 **Start/End 벡터 재사용**으로 메모리 효율화  
- **Coherency Vector** 추가로 “완전한 구문(Phrase)”만 필터링  
- Question은 **[CLS] 토큰 임베딩**으로 표현

### 🔸 Sparse Representation
- 문맥화된 임베딩을 사용해 **가장 관련성 높은 n-gram** 선택  
- Sparse Vector 구성 시 의미 손실 최소화

---

### ⚙️ Scalability Challenge
- Wikipedia 기준 **60B(600억 개) phrase** 존재 → 저장 및 검색 문제
- 해결 방법:
  - **Storage**: Pointer + Scalar Quantization (240TB → 1.4TB로 감소)
  - **Search**: **FAISS**로 Dense search 후 Sparse reranking 수행

---

### 🔸 DensePhrases (2021)
- **SpanBERT 두 개 사용**: 각각 Start/End 임베딩 계산  
- Sparse 임베딩 제거 → 저장 공간 절감  
- Retriever-reader 성능과 유사한 수준 도달

### 🔸 ColBERT (2020)
- **Late Interaction 방식**
  - Query와 문서를 **분리 인코딩** → 미리 문서 임베딩 구축 가능  
- **MaxSim 연산**으로 빠른 검색  
- 효율적이고 대규모 처리에 적합  

---

## 📊 3. Experiment Results & Analysis

### 🔸 DenSPI 결과 (SQuAD-open)
- Retriever-Reader보다 **+3.6% 정확도 향상**
- **68배 빠른 추론 속도 (CPU 기준 <1s)**  
- 단점:
  - 2TB SSD 필요 (저장 공간 큼)
  - **Decomposability Gap** 문제로 정확도 하락  
    - Question과 Answer가 분리 인코딩되며 Attention 손실  
    - BERT(90.9) → DenSPI(81.7)

### 🔸 DensePhrases 결과
- Decomposability Gap 해소  
- Sparse Embedding 제거로 **Storage 문제 완화**
- **Retriever-Reader 수준의 성능 회복**
- DPR과 비교 시:
  - **20배 빠른 속도**
  - **유사한 정확도** 확보  

---

## 🚀 정리 및 인사이트

- **Phrase Retrieval**은 ODQA의 속도와 효율성을 극대화한 방식  
- DensePhrases는 **성능·저장·속도** 모두 개선된 차세대 방식  
- **FAISS, BERT, SpanBERT, Dense/Sparse Hybrid** 등은  
  → 개인 프로젝트에서 **대용량 검색엔진 설계 또는 질의응답 시스템 구축 시** 핵심 참고 기술  

---
