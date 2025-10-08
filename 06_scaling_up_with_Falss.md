# 06강 | Passage Retrieval – Scaling Up with FAISS

> **핵심 주제:** Dense Embedding 기반 검색을 대규모로 확장하기 위한 효율적인 유사도 탐색 기법과 FAISS(Facebook AI Similarity Search)의 활용

## 📘 전체 요약

| 구분                 | 주요 개념                 | 핵심 목적                 |
| ------------------ | --------------------- | --------------------- |
| Passage Retrieval  | 질의-문서 유사도 기반 검색       | 관련 문서 선택              |
| MIPS               | 내적 기반 유사도 탐색          | Dense Embedding 검색 핵심 |
| Approximate Search | Compression + Pruning | 속도·정확도 균형             |
| FAISS              | 효율적 유사도 탐색 라이브러리      | 대규모 검색 최적화            |
| Scaling Up         | GPU, IVF-PQ           | 대규모 시스템 확장            |


---

## 1️⃣ Passage Search & Similarity Search

**문서 검색(Passage Retrieval)** 은 사용자의 질의(query)에 대해 가장 관련성 높은 문서를 찾는 과정입니다.
이 과정은 **Similarity Search** (유사도 검색) 문제로 표현할 수 있습니다.

### 💡 기본 개념

1. **Query 벡터화**

   * 질의 `q`를 인코딩하여 벡터 `h_q`로 변환
   * 질문의 의미를 수치 공간에서 표현

2. **Passage 벡터화**

   * 모든 문서를 인코딩하여 벡터 `h_p`로 변환
   * 각 문서는 하나의 벡터로 표현

3. **유사도 계산 (Similarity Scoring)**

   * `h_q`와 `h_p` 간의 내적(Dot Product) 또는 코사인 유사도 계산
   * 값이 높을수록 질의와 문서의 의미적 연관성이 큼

4. **랭킹 및 결과 반환**

   * 유사도 점수 순으로 정렬 → 상위 k개의 문서 선택

---

### ⚙️ MIPS (Maximum Inner Product Search)

* 주어진 질의 벡터 `q`와 내적값이 가장 큰 문서 벡터 `v`를 탐색
* Dense Embedding 기반 검색의 핵심 연산
* **Brute-force 방식:** 모든 벡터와 비교 (정확하지만 비효율적)

---

### ⚠️ 현실적인 문제: 대규모 코퍼스

* 예: 위키피디아(5백만 문서) → 수십억~수조 단위 확장 가능
* 모든 벡터를 직접 비교하면 **시간**, **메모리**, **정확도** 사이에 trade-off 발생

| 고려 요소        | 설명                   | 개선 방향                |
| ------------ | -------------------- | -------------------- |
| Search Speed | 검색 속도 (Query당 소요 시간) | Pruning (검색 공간 축소)   |
| Memory Usage | 메모리 사용량              | Compression (벡터 압축)  |
| Accuracy     | 검색 정확도               | Exhaustive Search 병행 |

> ✅ 핵심: **검색 속도와 재현율(Recall)은 반비례** 관계.
> 더 빠른 검색 = 더 작은 검색 공간 = 더 낮은 정확도.

---

## 2️⃣ Approximate Similarity Search

대규모 데이터셋에서의 효율적인 검색을 위해 **근사(Approximate)** 방식을 사용합니다.

---

### 📦 Compression – Scalar Quantization (SQ)

* 벡터의 각 요소를 더 작은 단위로 압축
* 예: 32비트 부동소수점 → 8비트 정수 변환
* 메모리 절약 효과 ↑, 정확도 ↓

> **압축(Compression):** 메모리 효율을 위해 일부 정보 손실을 허용하는 전략

---

### 🌲 Pruning – Inverted File (IVF)

* 전체 데이터 중 일부만 검색하도록 **검색 공간 축소**

#### 과정 요약

1. **Clustering:**

   * 전체 벡터 공간을 K개의 클러스터로 분할 (ex. K-means)
2. **Inverted Index 생성:**

   * 각 클러스터 중심(centroid)과 해당 벡터들을 연결
3. **검색 시:**

   * 질의 벡터 `h_q`와 가장 가까운 centroid 선택
   * 선택된 클러스터 내 벡터만 탐색

> 🎯 효과: 전체 데이터가 아닌 **일부 후보군만 탐색** → 속도 대폭 향상

---

## 3️⃣ Introduction to FAISS

**FAISS (Facebook AI Similarity Search)**
대규모 벡터 검색을 위한 고성능 오픈소스 라이브러리

* **공식 GitHub:** [facebookresearch/faiss](https://github.com/facebookresearch/faiss)
* **튜토리얼:** [FAISS Tutorial (Python)](https://github.com/facebookresearch/faiss/tree/main/tutorial/python)

### 주요 특징

* CPU/GPU 모두 지원
* Brute-force부터 Approximate Search까지 다양한 인덱스 구조 제공
* 수백만~수십억 개 벡터에서도 빠른 검색 가능

---

### 🔍 Passage Retrieval with FAISS

1. **인덱스 학습 (Train Index)**

   * 데이터 벡터를 기반으로 클러스터 중심을 학습
   * 예: `index.train(xb)`
2. **벡터 매핑 (Add Vectors)**

   * 학습된 인덱스에 벡터를 추가
3. **검색 수행 (Search)**

   * 쿼리 벡터 입력 후 유사도 높은 Top-k 벡터 반환
   * `nprobe`: 검색 시 방문할 클러스터 수 → 속도 vs 정확도 조절

```python
import faiss
index = faiss.IndexFlatL2(768)  # L2 거리 기반 인덱스
index.add(xb)                   # Passage 벡터 추가
D, I = index.search(xq, k=5)    # 질의 벡터 xq로 검색
```

---

## 4️⃣ Scaling Up with FAISS

### 🧱 주요 인덱스 유형

| 인덱스 종류                  | 설명                             | 특징                 |
| ----------------------- | ------------------------------ | ------------------ |
| **Flat (Brute-force)**  | 모든 벡터 비교                       | 가장 정확, 느림          |
| **IVF (Inverted File)** | 벡터 클러스터링 후 근접 클러스터만 탐색         | 빠름, 근사 결과          |
| **IVF-PQ**              | IVF + Product Quantization(압축) | 속도↑, 메모리↓, 정확도 일부↓ |

---

### ⚡ GPU 가속

* FAISS는 GPU 연산을 지원해 대규모 검색 속도를 획기적으로 개선
* GPU 행렬 연산을 이용해 벡터 간 거리 계산 병렬 처리
* 단점: GPU 메모리 한계 및 랜덤 접근 지연

### 💥 Multi-GPU 확장

* 여러 GPU를 병렬로 사용하여
  수십억 개 벡터를 동시에 검색 가능
* 대규모 Retrieval 시스템 구축에 적합

---

> ✅ **핵심 정리**
>
> * Dense Embedding을 실제 시스템에 적용하려면,
>   효율적 유사도 탐색 구조(FAISS 등)가 필수적이다.
> * 속도, 정확도, 메모리의 균형점을 설계하는 것이 핵심이다.
> * FAISS는 이러한 요구를 충족하는 대표적인 솔루션이다.
