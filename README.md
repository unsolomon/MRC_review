## 📚 **MRC (Machine Reading Comprehension) 요약 및 강의별 정리**

### 🧭 개요

이 저장소는 **기계독해(Machine Reading Comprehension, MRC)** 강의(1~10강)의 핵심 내용을 정리한 자료입니다.
각 강의는 **Open-Domain QA, Passage Retrieval, Dense/Sparse Embedding, Bias 문제, LLM 기반 QA** 등
MRC의 주요 개념과 실제 시스템 구현 흐름을 다루며 구성되어 있습니다.

---

## 📚 **강의별 정리**

| 강의      | 제목                                         | 주요 내용                                                     |
| ------- | ------------------------------------------ | --------------------------------------------------------- |
| **1강**  | 🧩 *MRC Introduction*                      | MRC 개념, 구성요소(Question, Context, Answer), 학습 구조 및 태스크 정의   |
| **2강**  | 🌝 *Extraction-based MRC*                  | 질문-문서 매칭 구조, Span Prediction, Start-End Position 학습       |
| **3강**  | 🤓 *Generation-based MRC*                  | 문장 생성 기반 QA, Seq2Seq 구조, T5/BART 모델 활용                    |
| **4강**  | 🔍 *Passage Retrieval - Sparse Embedding*  | TF-IDF, BM25 기반 Sparse Retrieval 기법                       |
| **5강**  | ⚙️ *Passage Retrieval - Dense Embedding*   | Dense Encoder, Negative Sampling, Embedding 학습 방식         |
| **6강**  | ⚡ *Passage Retrieval - Scaling Up (FAISS)* | FAISS를 활용한 대구모 검색, IVF & PQ 인덱스 구조                        |
| **7강**  | 🌐 *Linking MRC and Retrieval (ODQA)*      | ODQA 구조, Retriever-Reader 파이프라인, Distant Supervision      |
| **8강**  | 🧩 *Reducing Training Bias*                | Training/Annotation Bias 정의 및 완화 기법, Negative Sampling 전략 |
| **9강**  | 🤓 *Closed-book QA with LLMs*              | LLM 기반 Closed-book QA, LoRA 튜닝, Zero-shot/ICL 접근법         |
| **10강** | 💬 *QA with Phrase Retrieval*              | Phrase 단위 검색, DensePhrases & ColBERT 구조, 성능 분석            |

---

## 🧱 **MRC 시스템 구성 흐름**

graph TD
    A[질문 입력 (Query)] --> B[Passage Retrieval]
    B --> C[Dense/Sparse Embedding]
    C --> D[Retriever 결과 Top-k Passage]
    D --> E[Reader (MRC 모델)]
    E --> F[Answer Extraction or Generation]


* **Retriever**: 관련 문서를 빠르게 찾아주는 검색 모듈
* **Reader**: 검색된 문서에서 실제 답변을 예측하는 모델
* **ODQA(Open-Domain QA)**: 위 과정을 통합해 외부 지식 기반 질의응답 수행

---

## 🥉 **핵심 기술 요약**

| 기술명                           | 설명                             |
| ----------------------------- | ------------------------------ |
| **TF-IDF / BM25**             | Sparse Embedding 기반 검색 (희소 벡터) |
| **Dense Encoder (BERT, DPR)** | 의미 기반 벡터 검색                    |
| **FAISS**                     | 대구모 벡터 인덱스/검색 라이브러리            |
| **Negative Sampling**         | 학습 시 Hard Negative를 활용한 성능 향상  |
| **LoRA**                      | 대구모 언어모델의 효율적 Fine-tuning 기법   |
| **DensePhrases**              | 문구(Phrase) 단위의 검색 구조, 빠른 QA    |
| **ColBERT**                   | Query-Document 분리형 임벡딩 구조      |

---

## 🚀 **추천 학습 흐름**

1. **MRC의 기본 개념 이해** → (1~3강)
2. **Passage Retrieval 구조 학습** → (4~6강)
3. **Retrieval + Reader 통합 (ODQA)** → (7강)
4. **Bias 문제 인식 및 해결법 학습** → (8강)
5. **LLM을 활용한 QA 발전 방향** → (9~10강)

---

## 🤓 **추후 개인 프로젝트에 활용 예시**

* **Dense Retrieval 시스템 구현 (FAISS + DPR)**
* **Closed-book QA 실험 (T5 / Llama 기반)**
* **Phrase-level Retrieval 모델 (DensePhrases)**
* **LoRA 기반 LLM Fine-tuning 실시**

---

## 📂 **파일 구조 예시**

```
📁 MRC/
├── 01_MRC_Intro.md
├── 02_Extraction_based_MRC.md
├── 03_Generation_based_MRC.md
├── 04_Passage_Retrieval_Sparse.md
├── 05_Passage_Retrieval_Dense.md
├── 06_Passage_Retrieval_Scaling_Up.md
├── 07_Linking_MRC_and_Retrieval.md
├── 08_Reducing_Training_Bias.md
├── 09_Closed_book_QA_with_LLMs.md
├── 10_QA_with_Phrase_Retrieval.md
└── README.md
```

---

## ✍️ **참고**

* 본 문서는 강의 PDF 기반으로 **핵심 개념, 수식, 구조 및 실험 결과를 재구성한 학습 노트**입니다.
* 모든 자료는 학습 및 연구 목적으로만 사용해야 합니다.
