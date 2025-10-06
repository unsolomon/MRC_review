## ✅ Extraction-based MRC 핵심 요약

| 구분                  | 설명                                                          |
| ------------------- | ----------------------------------------------------------- |
| **정의**              | 질문의 답이 지문 내 특정 span에 존재                                     |
| **대표 데이터셋**         | SQuAD, KorQuAD, NewsQA, Natural Questions                   |
| **평가 방법**           | EM / F1 Score                                               |
| **전처리**             | Tokenization, Special Tokens, Attention Mask, Token Type ID |
| **학습(Fine-tuning)** | Start / End 위치 예측 (Token Classification)                    |
| **후처리**             | 불가능한 조합 제거, score 합산으로 최적 답 선택                              |

---

## 1. Extraction-based MRC

### 🧠 문제 정의

> 질문(question)의 답변(answer)이 항상 주어진 지문(context) 내에 **연속된 span(토큰의 구간)**으로 존재하는 형태의 MRC 문제.

즉, 모델은 **문서에서 정답이 위치한 구간의 시작(start)과 끝(end)을 예측**한다.

**대표 데이터셋**

* SQuAD
* KorQuAD
* NewsQA
* Natural Questions

이러한 데이터셋들은 모두 **Hugging Face Datasets** 라이브러리에서 접근 가능하다.
🔗 [https://huggingface.co/datasets](https://huggingface.co/datasets)

---

### ⚙️ 평가 방법 (Evaluation Metrics)

#### 🟩 Exact Match (EM)

* 예측값이 정답과 **문자(character) 단위로 완전히 일치할 때만 1점** 부여.
* 하나라도 다르면 0점 처리.

#### 🟨 F1 Score

* **예측값과 정답의 토큰 단위 overlap 비율**을 기준으로 0~1 사이의 점수를 부여.

| 항목            | 계산식                   | 예시                |
| ------------- | --------------------- | ----------------- |
| **Precision** | TP / (TP + FP)        | 1 / (1 + 0) = 1   |
| **Recall**    | TP / (TP + FN)        | 1 / (1 + 1) = 0.5 |
| **F1**        | 2 × (P × R) / (P + R) | ≈ 0.67            |

**예시 2**

* Precision = 2/3 ≈ 0.67
* Recall = 2/2 = 1
* F1 = 2×(0.67×1)/(0.67+1) = 0.8

🧩 **해석**

* **Precision(정밀도)**: 모델이 예측한 중 실제 정답 비율 (False Positive 최소화)
* **Recall(재현율)**: 실제 정답 중 모델이 맞힌 비율 (False Negative 최소화)
* **F1 Score**: Precision과 Recall의 균형을 나타냄.
  → **데이터 불균형 환경에서 특히 유용**

---

### 🔎 Overview

Extraction-based MRC의 전반적인 파이프라인:

1. **Pre-processing (전처리)**

   * 텍스트를 토큰화하고 모델 입력으로 변환
2. **Fine-tuning (미세조정)**

   * Pre-trained 모델(BERT 등)을 MRC 데이터셋에 맞게 학습
3. **Post-processing (후처리)**

   * start/end 위치 예측 결과에서 최적의 정답을 추출

---

## 2. Pre-processing (전처리)

전처리 과정은 **텍스트를 토큰 단위로 분해하고**,
모델이 이해할 수 있는 입력 형태로 만드는 과정이다.

---

### 🧩 Tokenization

> 텍스트를 작은 단위(Token)로 나누는 과정

* 기준: 띄어쓰기, 형태소, subword 등 다양함
* 최근에는 **BPE(Byte Pair Encoding)** 기반 토크나이저 사용
* 본 강의에서는 **WordPiece Tokenizer** 사용

#### 예시

“미국군대내두번째로높은직위는무엇인가?”
→ `[‘미국’, ‘군대’, ‘내’, ‘두번째’, ‘##로’, ‘높은’, ‘직’, ‘##위는’, ‘무엇인가’, ‘?’]`

---

### 🔖 Special Tokens

* `[CLS]`로 문장 시작
* `[SEP]`으로 **Question과 Context를 구분**

```
[CLS] Question [SEP] Context [SEP]
```

---

### 🧠 Attention Mask

* 모델이 **어떤 토큰에 주의(attention)**를 기울일지 결정
* 0 → 무시, 1 → 연산 포함
* 주로 `[PAD]` 같은 의미 없는 토큰을 무시하기 위해 사용

---

### 🧩 Token Type IDs

* 입력이 2개 이상의 시퀀스로 구성될 때(예: 질문 + 지문)
* 각 시퀀스에 **ID(0,1)**를 부여하여 모델이 구분 가능하도록 함

---

### 🧾 모델 출력값

* 정답은 **문서 내 연속된 span(토큰 구간)**
* 따라서 정답을 생성하지 않고,
  **“시작 위치(start)”와 “끝 위치(end)”를 예측**하는 **Token Classification 문제**로 변환.

즉, 모델은 다음 두 확률 분포를 학습한다:

* Start logits
* End logits

---

## 3. Fine-tuning (미세조정)

### ⚙️ 과정 요약

1. **Pre-trained Language Model (e.g., BERT)** 위에
   **start/end 위치 예측용 Linear Classifier**를 추가한다.
2. 각 토큰의 임베딩을 입력받아

   * Start 위치 확률 분포
   * End 위치 확률 분포
     를 출력한다.
3. **Cross Entropy Loss**로 학습.

### 🔁 목적

> 입력된 문장(context + question)에서 정답 span을 찾도록 모델을 최적화

---

## 4. Post-processing (후처리)

### 🚫 불가능한 답 제거하기

다음과 같은 경우 후보(candidates)에서 제외한다.

* End position이 start position보다 앞선 경우
  (예: start=90, end=80)
* 예측 위치가 context 범위를 벗어나는 경우
  (예: 질문 영역에 답이 존재)
* `max_answer_length`보다 긴 경우

---

### 🏆 최적의 답안 찾기

1. Start / End logits에서 **score가 높은 N개 후보**를 각각 선택
2. 불가능한 start-end 조합 제거
3. 가능한 조합들을 **score 합이 큰 순서로 정렬**
4. 최고 점수 조합을 최종 정답으로 선정
5. `Top-k`를 출력할 때는 상위 k개의 조합을 반환

---



📚 **참고문헌 및 출처**

* Rajpurkar et al., *SQuAD: 100,000+ Questions for Machine Comprehension of Text*
* Devlin et al., *BERT: Pre-training of Deep Bidirectional Transformers* (NAACL 2019)
* [https://huggingface.co/datasets/rajpurkar/squad](https://huggingface.co/datasets/rajpurkar/squad)
* [http://jalammar.github.io/illustrated-bert](http://jalammar.github.io/illustrated-bert)
* [https://huggingface.co/datasets](https://huggingface.co/datasets)
