# A Multi-Interval Transformer Model for Predicting Sepsis Mortality in the Intensive Care Unit

This repository contains the preprocessing code, time-series construction pipeline, and modeling notebooks used in the study  
**"중환자실 패혈증 사망 예측을 위한 다중 조합 시간격 트랜스포머 모델"**

본 연구에서는 MIMIC-IV 전자의무기록(EHR) 기반 시계열 데이터를 활용하여  
ICU 패혈증 환자의 사망 여부를 예측하는 **다중 조합 시간격 트랜스포머(Multi-Interval Transformer)** 모델을 제안합니다.

---

## 📌 Overview

Sepsis is one of the leading causes of ICU mortality, and early prediction is crucial for improving patient outcomes.  
기존 모델은 단일 시간대 데이터만 활용해 **시간적 패턴을 충분히 반영하지 못했다는 한계**가 존재합니다.

본 연구는 다음 네 가지 서로 다른 시간 간격별 시계열 데이터를 활용합니다:

- **6시간 간격(6h)**  
- **12시간 간격(12h)**  
- **24시간 간격(24h)**  
- **48시간 간격(48h)**  

각 시간격 시계열을 트랜스포머 인코더로 각각 인코딩한 후, 마지막 히든 벡터를 **결합(concatenate)**하여 예측을 수행합니다.
이는 다양한 시간 패턴을 결합함으로써 예측력을 대응적으로 증가시키는 방법입니다.  

---

## 📁 Repository Structure
전처리_모델링/
│
├── Data processing_0.ipynb
├── Data_processing_1.ipynb
├── merge_2.ipynb
├── modeling_3.ipynb
├── visualization_4.ipynb
│
└── .gitignore

**설명**  
- `Data processing_*.ipynb`: MIMIC-IV 원천 데이터 로딩 및 전처리  
- `merge_2.ipynb`: Lab / Procedure / Prescription 이벤트 통합  
- `modeling_3.ipynb`: Multi-Interval Transformer 구현  
- `visualization_4.ipynb`: 성능 비교 및 시각화  

---

## 🏥 Dataset Description (MIMIC-IV)
본 연구는 **MIMIC-IV (Medical Information Mart for Intensive Care IV)** 를 사용합니다.  
- ICU 입원: 94,458건  
- 기간: 2008–2019  
- 기관: Beth Israel Deaconess Medical Center  
- 패혈증 환자 필터링: ICD-10 기준  
- ICU 입원 기간 ≥ 3일  
- 최종 outcome: 퇴원일 기준 사망(1) / 회복(0) 
전체 6,781건의 패혈증 환자 중  
**18% 사망, 82% 회복 퇴원**.

---

## 📊 Multi-Interval Time-Series Construction
각 환자에 대해 퇴원일(D0) 기준 **전전날(D-2)** 시점을 기준으로 시계열을 구성합니다.
시간간격별 구성 예시 (각 10개 구간):

### ✔ 6시간 간격  
T6 = [(D-4.5, D-4.25), ..., (D-3.25, D-2)]

### ✔ 12시간 간격  
T12 = [(D-7, D-6.5), ..., (D-3.5, D-2)]

### ✔ 24시간 간격  
T24 = [(D-12, D-11), ..., (D-3, D-2)]

### ✔ 48시간 간격  
T48 = [(D-22, D-20), ..., (D-4, D-2)]

각 구간에서 Lab / Procedure / Prescription 의료 이벤트 발생 여부를 **이진 벡터(0/1)** 로 표현하여 시계열 행렬로 구성합니다.  

---

## 🧠 Model Architecture: Multi-Interval Transformer
제안 모델은 다음 네 개의 시간격 입력을 각각 독립적으로 Transformer Encoder에 통과시켜 히든 벡터를 생성합니다.
h6h = T2(T1(E(x6h)))
h12h = T2(T1(E(x12h)))
h24h = T2(T1(E(x24h)))
h48h = T2(T1(E(x48h)))

마지막 히든 벡터 4개를 concatenate하여 fully-connected layer로 전달:


구조 그림 전체는 논문 Figure 2 참고.  

---

## 🎯 Experiments & Settings

- Batch size: **8192**  
- Optimizer: **Adam**  
- Learning Rate: **1e-4 (scheduler 적용)**  
- Activation: **ReLU**  
- Regularization: BatchNorm, Dropout 0.5  
- Epochs: 최대 100 + Early Stopping  
- Evaluation: **10-fold Cross Validation**  
- Metrics: Precision / Recall / Accuracy / F1 / AUROC  

---

## 📈 Results
단일 시간격 모델 대비 **다중 조합 시간격 모델(M6h+12h+24h+48h)** 이  
모든 주요 지표(Recall 제외)에서 최고 성능을 기록했습니다.

### Performance Summary (from Table 1)
| Model | Precision | Recall | F1 | Accuracy | AUROC |
|-------|-----------|--------|----|----------|--------|
| M6h | 0.393 | 0.740 | 0.503 | 0.733 | 0.822 |
| M12h | 0.433 | 0.730 | 0.531 | 0.764 | 0.841 |
| M24h | 0.454 | 0.719 | 0.544 | 0.784 | 0.839 |
| M48h | 0.366 | 0.778 | 0.488 | 0.703 | 0.829 |
| M6h+12h | 0.463 | 0.753 | 0.561 | 0.783 | 0.854 |
| M6h+12h+24h | 0.498 | 0.725 | 0.581 | 0.809 | 0.859 |
| **M6h+12h+24h+48h** | **0.508** | 0.729 | **0.587** | **0.811** | **0.863** |

👉 **시간격을 많이 결합할수록 성능이 지속적으로 향상됨**  
👉 ICU 환자의 복잡한 시간적 변화 패턴을 더 잘 반영한다는 의미

---

## 🚀 Usage Guide

### 1. Preprocessing  
Data processing_0.ipynb
Data_processing_1.ipynb
merge_2.ipynb


### 2. Model Training 
modeling_3.ipynb


### 3. Visualization  
visualization_4.ipynb


---

## 📚 Citation
If you use this code, please cite:

---

## 📧 Contact
- Email: **gnldus812@gmail.com**  
- Author: **HuiYeon Jo**  
- Institution: DS&ML Center, The University of Suwon  
