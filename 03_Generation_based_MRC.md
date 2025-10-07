## ✅ Generation-based MRC 핵심 요약

| 단계           | 내용                            | 비고                        |
| ------------ | ----------------------------- | ------------------------- |
| **1. 정의**    | 답변을 직접 생성하는 MRC               | Seq2Seq 구조                |
| **2. 모델**    | BART / T5 / mT5               | Encoder–Decoder           |
| **3. 전처리**   | Tokenization + Special Tokens | `[SOS]`, `[EOS]`, `[PAD]` |
| **4. 학습 목표** | 텍스트 재구성 / 답변 생성               | Cross-Entropy Loss        |
| **5. 후처리**   | Beam Search로 최적 시퀀스 선택        | Auto-regressive           |
| **6. 평가**    | EM / F1 / BLEU / ROUGE        | 표현 다양성 고려                 |

---

## 1️⃣ Generation-based MRC 개요

### 🧠 문제 정의

MRC(Machine Reading Comprehension)는 주어진 **지문(Context)** 과 **질문(Question)** 을 기반으로 **답변(Answer)** 을 도출하는 과제다.

이를 푸는 접근은 크게 두 가지로 나뉜다.

| 방식                       | 설명                             | 유형                 | 대표 모델                 |
| ------------------------ | ------------------------------ | ------------------ | --------------------- |
| **Extraction-based MRC** | 정답이 지문 내 **일정 구간(span)** 으로 존재 | 분류(Classification) | BERT, ALBERT, ELECTRA |
| **Generation-based MRC** | 정답을 **지문을 보고 새롭게 생성**          | 생성(Generation)     | BART, T5, mT5         |

즉,

* **Extraction-based** → "지문 속 정답 위치를 찾아라"
* **Generation-based** → "지문과 질문을 보고 정답을 직접 말해라"

> ✅ *핵심:*
> Generation-based MRC는 “텍스트 생성” 문제로 접근하며,
> 모델이 정답을 직접 만들어내기 때문에 **자유도가 높지만 정답 평가가 어렵다.**

---

## 2️⃣ Generation-based MRC 평가 방법

Extraction 기반과 동일하게 **Exact Match (EM)** 와 **F1 Score** 를 사용한다.

* **EM (Exact Match)** : 예측 답이 정답과 완전히 동일한 경우만 1점
* **F1 Score** : 예측 답과 정답 간 **token overlap 비율**로 계산

> 서술형 답변의 다양성을 고려할 땐 **BLEU / ROUGE-L** 등의 생성 품질 지표도 활용 가능.

---

## 3️⃣ Generation-based MRC Overview

1. **Step 1 — Tokenization**

   * 지문과 질문을 토큰 단위로 분해
2. **Step 2 — Model Input**

   * 토큰화된 입력을 Generation 기반 MRC 모델(BART, T5 등)에 전달
3. **Step 3 — Answer Generation**

   * 모델이 답변 문장을 생성 (예: “미국 육군부참모총장”)

---

## 4️⃣ Extraction vs Generation 비교 요약

| 구분         | Extraction-based       | Generation-based       |
| ---------- | ---------------------- | ---------------------- |
| **문제 형태**  | 분류(Classification)     | 생성(Generation)         |
| **입출력 구조** | PLM + Classifier       | Seq2Seq PLM            |
| **정답 형태**  | 지문 내 span              | 자유 텍스트(Free-form text) |
| **출력 단위**  | start/end 위치           | 문장 전체 (토큰 시퀀스)         |
| **대표 모델**  | BERT, RoBERTa, ELECTRA | BART, T5, mT5          |
| **평가 지표**  | EM / F1                | EM / F1 / ROUGE / BLEU |
| **장점**     | 명확한 평가, 빠름             | 자연스러운 문장 생성, 범용성       |
| **단점**     | 정답이 지문 내 있어야 함         | 생성 다양성으로 평가 어려움        |

> 💡 *실전 Tip:*
>
> * **Extraction**은 “사실 기반 QA”, “KorQuAD” 등에 적합
> * **Generation**은 “설명형 QA”, “추론형 질의응답”, “대화형 MRC”에 적합

---

## 5️⃣ Pre-processing (입력 전처리)

### 🧾 데이터 예시

```json
{
 "question": "미국 군대 내 두 번째로 높은 직위는 무엇인가?",
 "answers": [{"answer_start": 204, "text": "미국 육군부참모총장"}],
 "id": "6521755-0-0"
}
```

* **Extraction-based MRC**: `answer_start` 위치를 사용해 span을 예측
* **Generation-based MRC**: 정답 **text만 있으면 충분**

---

### 🔠 Tokenization

> 텍스트를 의미 단위로 분리하고 인덱스로 변환하는 과정

* **WordPiece Tokenizer** 사용
* 사전에 등록된 단어 단위로 분리
* “##”은 **앞 단어에 이어지는 부분(subword)** 을 의미

예시:

```
문장: "미국군대내두번째로높은직위는무엇인가?"
토큰화: [‘미국’, ‘군대’, ‘내’, ‘두번째’, ‘##로’, ‘높은’, ‘직’, ‘##위는’, ‘무엇인가’, ‘?’]
인덱스 변환: [101, 23545, 8910, 2456, 5678, ...]
```

---

### 🧩 Special Tokens

> 모델이 입력 구조를 이해하도록 도와주는 특수 토큰

| Token             | 의미         | Generation-based MRC 사용 |
| ----------------- | ---------- | ----------------------- |
| **[CLS]**         | 문장 시작      | 선택적                     |
| **[SEP]**         | 문장 구분      | 선택적                     |
| **[PAD]**         | 시퀀스 길이 맞춤  | ✅ 필수                    |
| **[SOS] / [EOS]** | 문장 시작/끝 표시 | ✅ 사용 (디코더 입력용)          |

> 🧠 Extraction은 `[CLS]`, `[SEP]`, `[PAD]` 사용
> 🧠 Generation은 `[SOS]`, `[EOS]` 중심의 자연어 포맷으로 입력 구성

---

### 🧮 추가 입력 정보 (Additional Info)

| 항목                 | 역할           | Generation 기반 특징  |
| ------------------ | ------------ | ----------------- |
| **Attention Mask** | 연산할 토큰 여부 지정 | Extraction과 동일    |
| **Token Type IDs** | 입력 구분용 ID    | ❌ BART 등은 사용하지 않음 |
| **Padding**        | 시퀀스 길이 보정    | ✅ 필수              |

> ⚠️ BART는 BERT처럼 Segment 구분이 없어 `token_type_ids`가 필요하지 않음

---

### 📝 출력 표현

* **Extraction**: 답변의 시작/끝 위치(span index)
* **Generation**: **답변 전체 텍스트를 토큰 단위로 생성**
  → 각 step에서 모델이 다음 단어를 예측하는 **autoregressive decoding** 구조

---

## 6️⃣ Model (Generation-based PLMs)

### 🧱 BART (Bidirectional and Auto-Regressive Transformers)

> **BERT + GPT 하이브리드 구조**

| 구성                      | 특징                  |
| ----------------------- | ------------------- |
| **Encoder (BERT-like)** | 문맥을 양방향으로 인코딩       |
| **Decoder (GPT-like)**  | 한 방향으로 단어를 순차적으로 생성 |

**Pre-training Objective**

* 입력 텍스트를 손상시키고(마스킹, 단어 삭제, 순서 뒤섞기 등)
* 원래 문장을 복구하도록 학습 (denoising autoencoding)

**장점**

* MRC, 요약, 번역 등 다양한 **Seq2Seq task**에 사용 가능
* 인코더와 디코더를 동시에 활용하여 문맥 이해 + 생성 모두 가능

---

### ⚙️ BART 학습 개념 요약

| 단계               | 설명                      |
| ---------------- | ----------------------- |
| **1. Encoding**  | 손상된 문장을 인코더에 입력         |
| **2. Decoding**  | 원래 문장을 복원하도록 생성         |
| **3. Objective** | Reconstruction Loss 최소화 |
| **4. 결과**        | 문맥 이해 + 생성 능력 동시 확보     |

> 예: “그는 학교에 갔다.” → “그는 [MASK]에 갔다.”
> 모델이 ‘학교’를 올바르게 복원하도록 학습

---

### 🔥 다른 Generation 기반 모델들

| 모델                                         | 구조                    | 특징                           |
| ------------------------------------------ | --------------------- | ---------------------------- |
| **T5 (Text-to-Text Transfer Transformer)** | Encoder–Decoder       | 모든 NLP task를 “텍스트 → 텍스트”로 통일 |
| **mT5**                                    | 다국어 버전                | 100+개 언어 지원                  |
| **Flan-T5**                                | Instruction tuning 반영 | 명령 기반 질의응답에 강함               |

> 💡 **T5 vs BART**
>
> * T5는 task prefix(“question: … context: …”)를 붙여 학습
> * BART는 더 자유로운 포맷 지원

---

## 7️⃣ Post-processing (후처리)

### 🧭 Decoding

> 모델이 단어를 순차적으로 예측하면서 문장을 완성하는 과정

**Auto-regressive 구조:**
이전 step에서 생성된 토큰이 다음 step의 입력으로 사용됨.

* 첫 입력: `[SOS]` 또는 `<s>` (문장 시작 토큰)
* 종료 조건: `[EOS]` 생성 시 멈춤

---

### 🔍 Decoding 전략 (Searching Methods)

| 방법                    | 설명                     | 특징           |
| --------------------- | ---------------------- | ------------ |
| **Greedy Search**     | 각 단계에서 확률이 가장 높은 단어 선택 | 빠르지만 최적 보장 X |
| **Exhaustive Search** | 가능한 모든 조합 탐색           | 정확하지만 계산량 많음 |
| **Beam Search**       | 상위 k개 후보를 병렬 탐색        | 효율성과 품질 균형   |

> 💡 실전에서는 대부분 **Beam Search** 사용
> beam width는 3~5가 일반적

---



## 🚀 실무 및 프로젝트 활용 포인트

1. **KorQuAD** 등에서 *extractive → generative 변환 fine-tuning* 실험 가능
2. **BART-base 또는 T5-small**부터 시도 (학습 효율↑)
3. **prompt 포맷 설계**가 성능에 영향 (“질문: … 지문: … 답변: …”)
4. **Decoding 파라미터 (beam, temperature)** 조정으로 자연스러운 답 생성
5. 평가 시 **F1 + ROUGE-L** 병행 추천

---

📚 **참고**

* Lewis et al., *BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension* (ACL 2020)
* Raffel et al., *T5: Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer* (JMLR 2020)
* Hugging Face Transformers 문서: [https://huggingface.co/docs](https://huggingface.co/docs)
