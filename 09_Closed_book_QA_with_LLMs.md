## 📘 09_Closed_book_QA_with_LLMs.md

## 🧭 전체 요약

| 구분 | 핵심 내용 | 실무/프로젝트 적용 포인트 |
|------|------------|-----------------------------|
| Closed-book QA | 외부 문서 검색 없이 언어모델 자체의 지식으로 답변 | 사전학습 기반 질의응답 구현 시 적용 |
| LLM & Zero-shot | 학습 데이터 없이도 일반화된 문제 해결 가능 | 제로샷 QA, Few-shot Prompt 설계 참고 |
| LoRA | 효율적 Fine-tuning 기법 | GPU 메모리 제약 환경에서 LLM 튜닝 |
| 실험결과 | T5, LLaMA 기반 CBQA의 강점 및 한계 | Text-to-Text QA 파이프라인 설계 시 참고 |

---

## 1️⃣ Closed-book Question Answering (CBQA)

### 🔹 개념
- **Open-book QA**:  
  외부 지식(예: Wikipedia)을 문서 단위로 나누고 검색 후 답변 생성.  
  → 장점: 실시간 정보 반영 가능  
  → 단점: 검색 및 스토리지 비용이 큼  
- **Closed-book QA**:  
  모델이 **사전 학습 과정에서 이미 지식을 내재**하고 있다고 가정.  
  외부 검색 없이 모델 내부의 파라미터만으로 답변을 생성.

```text
Input:  "Who discovered penicillin?"
Output: "Alexander Fleming"
```

> ✅ 핵심:  
> Closed-book QA = “Retrieval-free knowledge utilization”

### 📚 실험적 근거
- 언어모델이 자체적으로 **사실 기반 지식(factual knowledge)** 을 보유한다는 연구 결과 존재 (예: Petroni et al., 2019)
- GPT-2, GPT-3 등은 **zero-shot QA**에서도 일정 수준의 정확한 답변을 생성 가능

---

## 2️⃣ Large Language Models & Zero-shot Performance

### ⚙️ Task-specific Fine-tuning의 한계
1. **라벨링 비용**이 높음  
2. **Pre-trained knowledge**가 손상될 위험 존재  
3. **Scaling Law**에 따라 모델 크기 확장이 어려움

→ 해결책:  
**“Zero-shot Learning + In-context Learning”**

### 💡 Text-to-Text Framework
모든 NLP task를 입력→출력의 **텍스트 변환 문제**로 통합  
예:  
- `translate English to Korean: hello` → “안녕”
- `question: What is AI?` → “Artificial Intelligence”

> 대표 모델: **T5 (Text-to-Text Transfer Transformer)**  
> - T5-base: 220M  
> - T5-XXL: 11B  
> - 다양한 NLP 문제를 단일 프레임워크로 처리 가능

### 🧠 LLM의 지식 학습 방식
- **SkipGram** / **BERT Masked LM** / **GPT Next-token prediction** / **T5 Span corruption**
- 결과적으로 모델이 파라미터 내부에 “세계 지식”을 저장

### 💬 In-context Learning
- 역전파 없이 **입력 내 예시(few-shot prompt)** 만으로 새로운 task 수행  
- GPT-3, Llama, ChatGPT 등은 이 능력을 기반으로 제로샷 QA 수행 가능

> “모델이 이미 세상을 배웠기 때문에, 새로운 예시만 주면 바로 이해한다.”

---

## 3️⃣ LoRA (Low-Rank Adaptation)

### 🧩 등장 배경
- LLM의 Fine-tuning 비용이 매우 크기 때문에, 전체 파라미터를 학습하지 않고 **일부만 조정**하는 효율적 기법 필요.

### 🔑 핵심 개념
| 구분 | 기존 Fine-tuning | LoRA |
|------|------------------|------|
| 학습 범위 | 전체 파라미터 | 일부 저차원 부분만 |
| 메모리 사용량 | 높음 | 매우 낮음 |
| 속도 | 느림 | 빠름 |
| 성능 | 유지 or 약간 향상 | 유지 or 개선 |

- LoRA는 `W ≈ W₀ + A·B` 형태로 가중치를 분해하여, **A, B만 학습**
- Hugging Face의 `peft` 라이브러리를 통해 쉽게 적용 가능

> ✅ LoRA = “대형 언어모델을 가볍게 미세 조정하는 기술”

---

## 4️⃣ Experiment Result & Analysis

### ⚙️ Experimental Setting
- **데이터셋**: Open-domain QA / MRC 데이터에서 Context 제거  
- **Salient Span Masking (SSM)**: 고유명사·날짜 등 의미 단위를 마스킹 후 학습  
- **Fine-tuning**: Pre-trained T5 모델로 추가 학습  
  - Input: `trivia question: <질문>`  
  - Output: `answer: <답변1> answer: <답변2>`

### 📊 결과 요약
| 모델 | 특징 | 성능 |
|------|------|------|
| **T5 (XXL)** | Context 없는 Closed-book QA | 높은 정확도 |
| **T5 + SSM** | 중요 토큰 마스킹 추가 | 성능 크게 향상 |
| **LLaMA (7B~75B)** | 소형 모델도 대형 LLM 성능 근접 | 효율적 구조 |

> Salient Span Masking 기법은 **지식 일반화(knowledge generalization)** 능력을 강화함.

### 🚨 False Negative 사례
1. **Phrasing mismatch** – 표현만 다름  
2. **Incomplete annotation** – 일부 정답 누락  
3. **Unanswerable** – 시점/맥락 의존적 질문

### ⚠️ 한계점
- 대형 모델일수록 속도·메모리 문제  
- 모델 내부 지식의 **출처 불명** (설명 가능성 낮음)  
- 새로운 지식 업데이트 어려움

---

### 🔗 Reference
- Raffel et al. (2020) *Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer*  
- Roberts et al. (2020) *How Much Knowledge Can You Pack Into the Parameters of a Language Model?*  
- Petroni et al. (2019) *Language Models as Knowledge Bases?*  
- Radford et al. (2019) *Language Models are Unsupervised Multitask Learners*

---
