## 1. MRC란 (Machine Reading Comprehension)

---

## ✅ 요약 정리

| 구분               | 핵심 내용                            |
| ---------------- | -------------------------------- |
| **MRC 정의**       | 지문과 질의를 바탕으로 답을 추론하는 문제          |
| **유형**           | 추출형 / 서술형 / 객관식                  |
| **주요 어려움**       | 의미 유사 문장, 상호 참조, 무응답 질문, 다중 홉 추론 |
| **평가 지표**        | EM, F1, ROUGE-L, BLEU            |
| **Unicode**      | 문자 → 코드포인트 매핑, UTF-8 인코딩 구조      |
| **Tokenization** | 단어/형태소/Subword 단위 분리, BPE 사용     |
| **Dataset**      | KorQuAD v1/v2, Hugging Face 지원   |

---



**기계독해(MRC)**란

> 주어진 지문(Context)을 이해하고, 주어진 질의(Query/Question)의 답변을 **추론하는 문제**를 의미한다.

### 💡 예시

* **질문(Query)**: “세계에서 가장 긴 강의 길이는 얼마인가?”
* **지문(Context)**: 위키피디아의 ‘아마존강’ 문단
* **답변(Answer)**: “약 6,400km”

MRC는 검색엔진, 대화시스템, 챗봇 등에서 핵심적으로 사용된다.

---

## 2. MRC의 종류

MRC 데이터셋은 **답변 형식**에 따라 세 가지로 분류된다.

### ① Extractive Answer Dataset (추출형)

* 답이 항상 지문(Context)의 일부분(segment/span)으로 존재한다.
* **예시**

  * Cloze Tests : CNN/DailyMail, CBT
  * Span Extraction : SQuAD, KorQuAD, NewsQA, Natural Questions
* **특징**

  * 모델은 지문에서 정답 구간을 정확히 “추출”해야 한다.

---

### ② Descriptive/Narrative Answer Dataset (서술형)

* 답이 지문에 그대로 존재하지 않고, **새롭게 생성된 문장** 형태로 표현된다.
* **예시**

  * MS MARCO, Narrative QA
* **특징**

  * 질의를 이해하고, 문맥을 기반으로 자연스러운 문장을 “생성”해야 한다.

---

### ③ Multiple-choice Dataset (객관식)

* 여러 후보(answer candidates) 중 하나를 선택하는 형태.
* **예시**

  * MCTest, RACE, ARC
* **특징**

  * 지문을 이해하고, 후보 중 가장 타당한 답을 선택해야 함.

---

### ✅ 정리

| 구분              | 설명                      | 대표 데이터셋        |
| --------------- | ----------------------- | -------------- |
| Extractive      | 지문 내에서 정답을 **추출**       | SQuAD, KorQuAD |
| Descriptive     | 지문 내용을 기반으로 **새 문장 생성** | MS MARCO       |
| Multiple-choice | 후보 중 **하나 선택**          | RACE, ARC      |

---

## 3. MRC의 어려운 점 (Challenges in MRC)

MRC 모델이 실제로 어려움을 겪는 대표적인 이유들이다.

| 유형                         | 설명                                    | 예시 데이터셋            |
| -------------------------- | ------------------------------------- | ------------------ |
| **Paraphrasing**           | 단어 구성이 다르지만 의미가 같은 문장을 이해해야 함         | DuoRC              |
| **Coreference Resolution** | ‘그’, ‘그녀’, ‘그것’과 같은 대명사가 무엇을 지칭하는지 파악 | QuoRef             |
| **Unanswerable Questions** | 지문 내에 답이 존재하지 않는 경우                   | SQuAD 2.0          |
| **Multi-hop Reasoning**    | 여러 문서에서 정보를 종합해야 답을 찾을 수 있음           | HotpotQA, QAngaroo |

🧩 **요약**

> * 동일 의미 다른 단어
> * 상호참조(그/그녀 등)
> * 답이 없는 질문
> * 여러 문서를 넘나드는 추론 과정

---

## 4. MRC의 평가 방법 (Evaluation Metrics)

### ① Exact Match (EM) / Accuracy

* 예측 답이 정답과 **완전히 일치**하는 비율.
* 예:

  * Prediction: “for 5 days”
  * Ground-truth: “5 days”
  * → EM: 0 (불일치)

---

### ② F1 Score

* **Token 단위로 겹치는 비율(Token Overlap)**을 기준으로 평가.
* 예:

  * Ground-truth: “5 days”
  * Prediction: “for 5 days”
  * Overlap: “5”, “days” → F1 ≈ 0.8

---

### ③ ROUGE-L / BLEU

* **서술형 답변 데이터셋**에 적합.

| Metric      | 기준        | 설명                                  |
| ----------- | --------- | ----------------------------------- |
| **ROUGE-L** | Recall    | 정답과 예측 간 **순서가 유지된 공통 부분**(LCS)을 비교 |
| **BLEU**    | Precision | 문장 간 **n-gram 일치율**로 유사도 측정         |

📌 정리:

* **Extractive/객관식** → EM, F1 사용
* **서술형** → ROUGE-L, BLEU 사용

---

## 5. Unicode & Tokenization

### 🔤 Unicode란

> 전 세계의 모든 문자를 **일관되게 표현**하고 다룰 수 있도록 만든 문자셋.

* 각 문자를 **숫자(코드포인트)**에 매핑한다.
* **인코딩(Encoding)**: 문자를 컴퓨터가 저장할 수 있게 이진수로 변환.

#### UTF-8 인코딩 구조

| Byte 수  | 문자 예시                            |
| ------- | -------------------------------- |
| 1 byte  | Standard ASCII                   |
| 2 bytes | Arabic, Hebrew, European scripts |
| 3 bytes | BMP (한글 포함)                      |
| 4 bytes | Emoji 등 전체 유니코드 문자               |

#### Python에서 다루기

```python
ord('A')     # → 65
ord('가')    # → 44032
chr(65)      # → 'A'
chr(44032)   # → '가'
```

#### 한국어와 Unicode

* **완성형**: '가', '각' 등 모든 한글 조합(11,172자, U+AC00~U+D7A3)
* **조합형**: 초성/중성/종성을 조합해 문자를 형성

---

### ✂️ Tokenization (토크나이징)

> 텍스트를 **토큰(token)** 단위로 나누는 과정.

* 단어, 형태소, **subword** 등 다양한 단위로 가능.

#### Subword Tokenization

* 자주 등장하는 글자 조합은 하나의 단위로 취급.
* “##” 기호는 **앞 단어에 이어 붙임**을 의미.

#### BPE (Byte Pair Encoding)

* 원래는 **데이터 압축 알고리즘**, NLP에서 토크나이징에 응용.

1. 가장 자주 등장하는 글자 쌍(Bigram)을 새로운 토큰으로 병합
2. 병합 정보를 사전에 저장
3. 이 과정을 반복 수행

---


### 📂 Dataset: KorQuAD (Korean QA Dataset) 데이터 접근
> **LG CNS**가 공개한 한국어 질의응답 / 기계독해 데이터셋
* Hugging Face Datasets 라이브러리를 통해 로드 가능:

```python
from datasets import load_dataset
dataset = load_dataset("squad_kor_v1")
```

🔗 [HuggingFace KorQuAD](https://huggingface.co/datasets?search=korquad)


📚 **참고문헌 및 출처**

* Liu et al., *Neural Machine Reading Comprehension: Methods and Trends* (2019)
* Bajaj et al., *MS MARCO* (2016)
* Rajpurkar et al., *SQuAD 2.0* (2018)
* Yang et al., *HotpotQA* (2018)
* Kim et al., *KorQuAD 2.0: 한국어 질의응답 데이터셋* (2019)
* [HuggingFace Datasets Docs](https://huggingface.co/docs/datasets)
* [KorQuAD 공식 사이트](https://korquad.github.io/)
