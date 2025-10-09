# 08. Reducing Training Bias

> **목표:**  
> ODQA(Open-domain Question Answering) 시스템에서 발생하는 다양한 **편향(Bias)** 을 이해


## 🧩 전체 요약

| 구분 | 내용 | 핵심 키워드 |
|------|------|--------------|
| **Bias 정의** | 학습/데이터/어노테이션 단계의 왜곡 현상 | Inductive, Sampling, Annotator |
| **ODQA 내 Bias** | Reader가 정답만 포함된 문서에 노출 → 일반화 저하 | Negative 학습, No-answer bias |
| **Annotation Bias** | MRC 데이터를 그대로 사용 시 ODQA에 부적합 | Natural Questions 기반 수집 |


---

## 1️⃣ Definition of Bias

### 📘 Bias의 정의
Bias란 모델이 학습하거나 예측하는 과정에서 **특정 방향으로 치우친 경향**을 보이는 현상입니다.  
이는 데이터, 학습 구조, 어노테이션 과정 등 여러 요인에서 발생할 수 있습니다.

---

### 📊 Bias의 주요 유형

| 구분 | 설명 | 예시 |
|------|------|------|
| **Inductive Bias** | 학습 시 과적합을 방지하거나 사전 지식을 주입하기 위해 특정 함수 형태를 선호 | 정규화, 모델 구조 설계 등 |
| **Historical Bias** | 현실 세계의 편향이 그대로 반영된 경우 | 성별–직업 관계 고착 |
| **Co-occurrence Bias** | 자주 함께 등장하는 단어의 상관관계를 모델이 잘못 학습 | “요리사=여성”, “의사=남성” 등 |
| **Specification Bias** | 입력/출력 정의 과정에서 발생하는 구조적 편향 | 문서 제한, 불균형 데이터 정의 |
| **Sampling Bias** | 데이터 수집 과정에서 특정 집단만 샘플링 | 일부 계층 여론만 반영된 설문 |
| **Annotator Bias** | 주석자(Annotator)의 주관적 판단이 데이터에 반영 | 감성 라벨링의 개인차 등 |

> 💡 **핵심 인사이트:**  
> Bias는 단순히 “데이터의 불균형”이 아니라,  
> “모델이 세상을 왜곡되게 해석하게 만드는 모든 요인”을 의미한다.

---

## 2️⃣ Bias in Open-domain Question Answering

### 🔍 ODQA의 구조
ODQA는 **Retriever–Reader Pipeline**으로 구성됩니다.

| 구성 요소 | 역할 |
|------------|------|
| **Retriever** | 질문과 관련 있는 문서 검색 |
| **Reader** | 검색된 문서 내에서 정답 추론 |

---

### ⚠️ 문제점: Training Bias in Reader
Reader 모델이 **항상 정답이 포함된 문서(Context)** 로만 학습될 경우,  
“정답이 없는 문서(No-answer context)”를 처리하지 못함.

#### 예시:
SQuAD처럼 `(Context, Query, Answer)`가 항상 존재하는 데이터로만 학습한 모델은  
새로운 문서에서 답변이 없을 경우 **무조건 오답을 출력**하게 됨.

---

### ✅ 해결 방안

#### 1️⃣ Train Negative Examples
- 훈련 시 “정답이 없는 문서(negative samples)”를 함께 학습시킴.
- 모델이 올바른 문서와 잘못된 문서를 **벡터 공간상에서 분리**하도록 유도.

| Negative 유형 | 설명 |
|----------------|------|
| **Random Negative** | 코퍼스 내 임의의 문서 선택 |
| **Hard Negative** | 높은 BM25/TF-IDF 점수를 가지지만 정답이 포함되지 않은 문서 |
| **In-document Negative** | 동일 문서 내 다른 Passage를 Negative로 사용 |

> → 다양한 난이도의 Negative를 포함해야 일반화가 향상됨.

---

#### 2️⃣ Add No-Answer Bias
- 입력 시퀀스 길이 `N` 외에 **추가 토큰(“no answer”)** 을 둠.  
- Softmax 계산 시, 해당 위치의 확률이 가장 높으면 “대답할 수 없음”으로 처리.  
- 모델의 마지막 레이어에 학습 가능한 **bias weight** 를 추가하여 구현.

> ⚙️ 예시:  
> Start/End 예측 외에 bias 노드 추가 →  
> 해당 위치의 확률이 최댓값일 경우 → “답변 없음” 판정

---

## 3️⃣ Annotation Bias from Datasets

### 💬 개념
MRC 데이터셋(SQuAD, KorQuAD 등)을 ODQA 학습에 그대로 사용하면,  
데이터 제작(어노테이션) 과정에서 **ODQA 환경과 맞지 않는 편향**이 포함될 수 있습니다.

---

### ⚠️ 주요 원인

| 원인 | 설명 |
|------|------|
| **정보 탐색형 질문 부재** | 실제 사용자는 답을 모르고 질문하지만, MRC 데이터는 답을 알고 작성 |
| **질문–문단 단어 중복** | 질문자와 어노테이터가 동일해 단어가 과도하게 겹침 |
| **제한된 문서 출처** | SQuAD: 약 500개 위키 문서에 집중되어 있음 |
| **비현실적 분포** | 실제 검색 로그와 다른 구조의 질의/문서 비율 |

---

### 🧠 영향
- Sparse 기반(BM25)과 Dense 기반(DPR) 모델 간 성능 차이 발생  
- Reader가 “정보 탐색형 질의”에 취약해짐  
- 새로운 도메인(뉴스, 커뮤니티 등)에서 일반화 성능 저하

---

### ✅ 개선 방향

- **ODQA와 유사한 환경의 데이터 구성**
  - 실제 사용자 질의 기반 데이터 수집 (예: Google *Natural Questions*)  
  - Supporting Evidence 없이 질문만 제공  
  - Annotator가 “검색을 통해” 답을 찾아야 하는 구조 설계

- **Annotation 단계에서의 인지적 보정**
  - 질문 작성자와 검수자 분리  
  - 문서/질문/답변 간 단어 중복률 제한  
  - 데이터 검증 단계에서 bias 점검 자동화

---


### ✍️ Study Notes
- Reader 모델은 “정답이 없는 문서”를 학습해야 실제 환경에 강함.  
- Hard Negative 샘플을 충분히 확보하는 것이 중요.  
- 어노테이션 편향은 모델 구조보다 데이터 수집 설계에서 먼저 해결해야 함.  
- Bias 완화는 단순한 정규화가 아니라 **데이터+모델+훈련 세트 설계의 통합적 접근**이 필요하다.

