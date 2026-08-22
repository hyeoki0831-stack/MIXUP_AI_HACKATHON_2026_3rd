# MIXUP_AI_HACKATHON_2026_3rd
MIXUP AI HACKATHON 2026 3rd Track 2

2026년 5월에 참여한 "MIXUP AI HACKATHON 2026 3rd Track 2" 대회 정리 및 기록 문서입니다.
3명의 팀원을 포함한 하나의 팀으로 대회에 참가하였습니다.

대회 개요
--------------------
- 대회 명:MIXUP AI HACKATHON 2026 3rd track2
- 대회 일정:2026.05.22(금) 20:00~ 2026.05.23(토) 13:00
- 주최 동아리:BITAmin, Prometheus, TOBIG's
- 주관: Upstage, KRIBB
- 대회 링크:https://dacon.io/competitions/official/236720/overview/description (track 2)
- 문제 유형
  - track 1: [업스테이지]SOLAR PRO3를 활용한 Agent 개발
  - track 2: [한국생명공학연구원]분자 구조 기반 hERG inhibition 회귀 예측 모델 개발

문제 주제 및 데이터 설명
-------
**문제 주제: 분자 구조 기반 hERG inhibition at 1µM 회귀 예측 AI 모델 개발**

- **평가 지표:** MAE (Mean Absolute Error)
MAE는 실제값과 예측값 사이의 절대 오차 평균이고, 값이 낮을수록 좋은 성능을 의미함.

**데이터 설명**

각 화합물은 기본적인 `SMILES` 정보와 함께,
분자 구조를 서로 다른 방식으로 표현한 세 종류의 feature로 제공됩니다.

- **Train:** 39,994개 화합물
- **Test:** 9,998개 화합물
- **Target:** `hERG_inhibition`
- `hERG_inhibition`은 **1µM 농도에서 측정된 hERG percent inhibition 값**입니다.

| Feature 종류 | 차원 | 설명 |
|---|---:|---|
| **RDKit Descriptor** | 210 | 분자의 물리화학적·구조적 특성을 수치화한 변수 |
| **Morgan Fingerprint** | 2,048 | 분자의 국소적인 부분구조 패턴을 0/1 bit로 표현 |
| **ChemBERTa Embedding** | 768 | SMILES를 Transformer가 학습하여 생성한 잠재 표현 |
| **전체 Feature** | **3,026** | 위 세 종류의 feature를 모두 결합 |


### 주요 변수 설명

| 변수 그룹 | 주요 변수명 | 설명 |
|---|---|---|
| **식별자** | `id` | 각 화합물 샘플을 구분하기 위한 고유 ID |
| **분자 구조** | `smiles` | 화합물의 분자 구조를 문자열 형태로 표현한 SMILES |
| **타깃 변수** | `hERG_inhibition` | 1 µM 농도에서 측정된 hERG percent inhibition 값 |
| **분자 크기·질량** | `MolWt`, `ExactMolWt`, `HeavyAtomCount` | 분자의 질량과 크기를 나타내는 특성 |
| **지용성·극성** | `MolLogP`, `TPSA` | 분자의 지용성과 극성에 관련된 물리화학적 특성 |
| **수소결합 특성** | `NumHDonors`, `NumHAcceptors` | 수소결합 공여체와 수용체의 수 |
| **구조적 유연성** | `NumRotatableBonds`, `FractionCSP3` | 분자의 회전 가능한 결합 및 3차원적 구조 특성 |
| **고리 구조** | `RingCount`, `NumAromaticRings` | 분자 내 고리 및 방향족 고리의 수 |
| **표면·전자적 특성** | `PEOE_VSA*`, `SlogP_VSA*`, `EState_VSA*` | 부분전하, 지용성, 전자적 상태와 표면적을 결합한 Descriptor |
| **부분구조** | `fr_*` | 특정 작용기 또는 부분구조의 개수를 나타내는 Descriptor |
| **구조 패턴** | `morgan_r2_bit_0` ~ `morgan_r2_bit_2047` | Radius 2 Morgan fingerprint의 2,048차원 이진 feature |
| **잠재 분자 표현** | `chemberta_0` ~ `chemberta_767` | ChemBERTa가 SMILES로부터 학습한 768차원 latent embedding |

### 데이터 파일

#### Train

| 파일 | 포함 정보 |
|---|---|
| `train/train.csv` | `id`, `smiles`, `hERG_inhibition` |
| `train/train_rdkit_descriptor.csv` | 기본 변수 + RDKit Descriptor 210개 |
| `train/train_rdkit_fingerprint.csv` | 기본 변수 + Morgan Fingerprint 2,048 bit |
| `train/train_chemberta_feature.csv` | 기본 변수 + ChemBERTa Embedding 768차원 |
| `train/train_all_features.csv` | Descriptor + Fingerprint + ChemBERTa를 모두 결합한 데이터 |

#### Test

| 파일 | 포함 정보 |
|---|---|
| `test/test.csv` | `id`, `smiles` |
| `test/test_rdkit_descriptor.csv` | 기본 변수 + RDKit Descriptor 210개 |
| `test/test_rdkit_fingerprint.csv` | 기본 변수 + Morgan Fingerprint 2,048 bit |
| `test/test_chemberta_feature.csv` | 기본 변수 + ChemBERTa Embedding 768차원 |
| `test/test_all_feature.csv` | Descriptor + Fingerprint + ChemBERTa를 모두 결합한 데이터 |

### 분자 표현 Feature

하나의 화합물을 서로 다른 방식으로 표현한 세 종류의 feature가 제공됩니다.

#### 1. RDKit Descriptor

SMILES로부터 계산한 분자의 물리·화학적 및 구조적 특성임.

총 210개의 feature가 제공되며, 대표적으로 다음과 같은 정보가 있음.

- 분자량 (Molecular Weight)
- LogP
- TPSA
- 수소결합 Donor / Acceptor
- Rotatable Bond
- Ring 관련 특성

#### 2. Morgan Fingerprint

분자 내부의 국소적인 구조 및 부분구조 정보를 고정 길이의 이진 벡터로 표현한 데이터입니다.

- Radius: 2
- Dimension:2,048

원자 주변의 부분구조 정보를 해시하여 각 fingerprint bit의 활성 여부로 표현함.

#### 3. ChemBERTa Feature

대회에서 제공된 seyonec/ChemBERTa-zinc-base-v1 기반의 SMILES embedding으로, 총 768차원의 연속형 feature로 구성되어 있음.

- Dimension: **768**


데이터 분석
-------
## 주요 점검 항목 및 EDA

대회 종료 후 프로젝트를 정리하는 과정에서 원본 데이터를 다시 점검하고,
추가적인 탐색적 데이터 분석(EDA)을 수행했습니다.

**주요점검항목**

| 점검 항목 | 확인 목적 |
|---|---|
| **Target 분포** | 치우침, 음수값, 극단값 등 회귀 문제의 특성 파악 |
| **결측치 / 무한값** | 전처리가 필요한 비정상 값 존재 여부 확인 |
| **중복 데이터** | 동일 화합물 또는 중복 행으로 인한 데이터 누수 가능성 확인 |
| **상수 / Near-constant 변수** | 정보량이 거의 없는 feature 확인 |
| **Feature scale / 이상값** | 비정상적으로 큰 값이나 심한 왜도를 가진 변수 확인 |
| **Fingerprint 희소성** | 2,048차원 이진 feature의 실제 활성화 정도 확인 |
| **Train / Test 분포 차이** | 학습 데이터와 평가 데이터 간 distribution shift 가능성 확인 |
| **SMILES 길이** | 비정상적으로 길거나 특이한 분자 표현 존재 여부 확인 |

### 결측값 및 데이터 품질

- **결측치: 0개**
- **`+inf`, `-inf`: 0개**
- **중복 `id`: 0개**
- **중복 `smiles`: 0개**
- **완전히 동일한 행: 0개**

### Target 분포 및 이상치

| 지표 | 값 |
|---|---:|
| 평균 | 2.3985 |
| 중앙값 | 6.8613 |
| 표준편차 | 17.0614 |
| 최솟값 | -118.7569 |
| 최댓값 | 126.4892 |
| 음수 비율 | 29.24% |
| IQR 기준 Outlier 비율 | 8.47% |
| 왜도 | -2.3810 |

Target의 약 **29%가 음수**였으며,
평균보다 중앙값이 높고 왜도가 음수로 나타나
**음수 방향으로 긴 꼬리를 가진 좌편향 분포**임을 확인하였음.

### RDKit Descriptor 점검

총 210개의 RDKit Descriptor 중 일부는 데이터 내 변화가 거의 없는 변수였습니다.

- **상수 변수:** 7개
- **Near-constant 변수:** 21개
- **고유값이 5개 이하인 변수:** 68개

예를 들어 `NumRadicalElectrons`, `SMR_VSA8`, `SlogP_VSA9` 등은
Train 데이터에서 값의 변화가 없는 상수 변수로 확인되었음.

### Descriptor와 Target의 관계

각 RDKit Descriptor와 `hERG_inhibition` 사이의
Pearson 및 Spearman 상관관계를 확인했습니다.

Pearson 절댓값 기준 상위 변수는 다음과 같았습니다.

| Descriptor | Pearson r |
|---|---:|
| `EState_VSA4` | -0.2125 |
| `PEOE_VSA8` | -0.2102 |
| `Chi3n` | -0.2088 |
| `Chi1n` | -0.2076 |
| `Chi4n` | -0.1975 |

가장 높은 절대 상관도 약 0.21 수준으로,
단일 Descriptor의 선형 관계만으로 Target을 설명하는 데에는 한계가 있음을 확인했음.

이는 여러 분자 특성을 함께 고려하는 모델링이 필요할 가능성을 보여준다.

### Morgan Fingerprint 희소성

2,048차원의 Morgan Fingerprint는 대부분의 bit가 0인
매우 희소한(sparse) 데이터였음.

- Train 전체 bit 중 `1`: **2.25%**
- Test 전체 bit 중 `1`: **2.24%**
- 화합물당 평균 활성 bit: **약 46개**
- 화합물당 중앙 활성 bit: **46개**

즉 하나의 화합물에서는 평균적으로 `2,048개 중 약 46개의 bit만 활성화`되어 있었음.

### ChemBERTa Feature 점검

ChemBERTa Embedding 768개 차원은 모두 연속형 값으로 구성되어 있었으며,

- 상수 dimension: **0개**
- Near-constant dimension: **0개**

로 확인되었음.

### Train / Test 분포 점검

학습 데이터의 패턴이 Test에도 동일하게 적용될 수 있는지 확인하기 위해
Train과 Test의 feature 분포를 비교했습니다.

개별 feature의 표준화 평균차(SMD)를 확인한 결과,

- RDKit Descriptor 최대 `|SMD|`: 약 **0.055**
- Morgan Fingerprint 최대 `|SMD|`: 약 **0.068**
- ChemBERTa 최대 `|SMD|`: 약 **0.068**

로 나타났습니다.

단일 feature 기준으로 매우 큰 분포 차이는 확인되지 않았지만,
일부 변수에서는 상대적으로 Train/Test 차이가 존재했습니다.

RDKit Descriptor 중 차이가 상대적으로 크게 나타난 변수는

- `SMR_VSA4`
- `NumAromaticRings`
- `SlogP_VSA8`
- `PEOE_VSA14`

등이었습니다.

단일 Feature 기준으로 매우 큰 분포 차이는 확인되지 않았지만,
이러한 단변량 비교만으로 Train과 Test가 동일한 분포라고 단정할 수 없음.

###  EDA를 통해 얻은 주요 시사점

1. **결측치와 중복 문제는 크지 않았다.**
   - 데이터 정제보다 feature 특성과 Target 분포를 이해하는 것이 더 중요했습니다.

2. **Target은 일반적인 0~100 범위의 단순 비율 데이터가 아니었다.**
   - 음수값과 100을 초과하는 값이 실제로 존재함. 음수도 의미있는 값일 수 있음.

3. **일부 RDKit Descriptor는 정보량이 매우 적었다.**
   - 상수 및 near-constant 변수가 존재해 feature selection 가능성을 확인했음.

4. **단일 Descriptor와 Target의 관계는 강하지 않았다.**
   - 개별 feature보다는 여러 분자 특성의 비선형 관계를 학습할 필요가 있었음.

5. **Morgan Fingerprint는 매우 희소한 고차원 데이터였다.**

   - Descriptor와 다른 형태로 분자의 부분구조 정보를 표현한다는 특징을 확인했습니다.

6. **ChemBERTa Embedding은 768개 차원 모두에서 변동성이 존재했다.**
   - 사람이 직접 의미를 해석하기 어려운 latent representation이지만,
     Descriptor나 Fingerprint와는 다른 방식으로 SMILES 정보를 표현합니다.

7. **Train/Test의 단변량 Feature 분포에서 큰 차이는 확인되지 않았다.**

   - 다만 일부 Feature에서 상대적인 차이가 존재했으며,
     단변량 분석만으로 전체적인 distribution shift를 판단하는 데에는 한계가 있습니다.




```text
EDA
 ↓
최종 모델 구성
 ↓
Base Model 10개
 ↓
Meta Feature 구성
 ↓
LGBM / Huber Stacking
 ↓
7:3 Final Blend
 ↓
최종 성능
```

## 🤖 모델링 및 실험

### 1. 최종 모델 구성

최종 제출 모델은 여러 Base Model의 예측값을 다시 Meta Model에 입력하는
**Stacking Ensemble** 구조로 구성했습니다.

최종 Meta Model의 입력에는

- **10개의 Base Model prediction**
- **10개의 물리화학적 Feature**

를 사용했습니다.

전체 구조는 다음과 같습니다.

```text
10 Base Model Predictions
          +
10 Physicochemical Features
          ↓
 ┌────────┴─────────┐
 ↓                  ↓
LGBM L1            Huber
Meta Learner       Regression
OOF 8.3807         OOF 8.4379
 └────────┬─────────┘
          ↓
0.7 × LGBM + 0.3 × Huber
          ↓
    OOF MAE 8.3773
    LB MAE  8.5201
```

---

### 2. Base Models

최종 Stacking에는 총 **10개의 Base Model prediction**을 사용했습니다.

| Base Model | OOF MAE |
|---|---:|
| `chemprop_bag_all` | **8.6065** |
| `team1_pseudo_v3` | **8.7940** |
| `chemprop_dmpnn` | **8.8321** |
| `team1_bag` | **8.8654** |
| `lgbm_descriptor_plus_bag` | 9.1281 |
| `lgbm_descriptor_plus` | 9.1652 |
| `dplus_L1_full` | 9.1711 |
| `dplus_MSE_full` | 9.3104 |
| `lgbm_fingerprint` | 9.3574 |
| `chemberta_v3` | 9.4748 |

최종 Ensemble에는 Chemprop 계열, LightGBM 계열,
Fingerprint 기반 모델, ChemBERTa 기반 모델 등
서로 다른 형태의 Base Model prediction이 포함되었습니다.

---

### 3. Meta Feature 구성

10개의 Base Model prediction에
다음 **10개의 물리화학적 Feature**를 추가하여 Meta Model의 입력으로 사용했습니다.

| Feature | 의미 |
|---|---|
| `LogP` | 지용성 |
| `MolWt` | 분자량 |
| `TPSA` | 위상학적 극성 표면적 |
| `HBD` | Hydrogen Bond Donor 수 |
| `HBA` | Hydrogen Bond Acceptor 수 |
| `Rotatable Bonds` | 회전 가능한 결합 수 |
| `Aromatic Rings` | 방향족 고리 수 |
| `fr_aniline` | Aniline 구조 관련 Descriptor |
| `fr_piperdine` | Piperidine 구조 관련 Descriptor |
| `fr_pyridine` | Pyridine 구조 관련 Descriptor |

따라서 최종 Meta Model은 총 **20개의 입력 Feature**를 사용했습니다.

```text
10 Base Predictions
       +
10 Physicochemical Features
       ↓
20 Meta Features
```

---

### 4. LGBM Meta Learner

첫 번째 Meta Model로 **LightGBM의 L1 Regression**을 사용했습니다.

10개의 Base prediction과 10개의 물리화학적 Feature를 입력으로 받아
최종 Target을 예측하도록 학습했습니다.

**OOF MAE: 8.3807**

---

### 5. Huber Meta Learner

두 번째 Meta Model로 **Huber Regression**을 사용했습니다.

LGBM Meta Model과 동일하게

- 10개 Base Model prediction
- 10개 물리화학적 Feature

를 입력으로 사용했습니다.

**OOF MAE: 8.4379**

---

### 6. Final Blend

두 Meta Model의 prediction을 최종적으로 다시 결합했습니다.

**Final Prediction = 0.7 × LGBM + 0.3 × Huber**

즉,

- **LGBM Meta Learner: 70%**
- **Huber Regression: 30%**

의 비율로 Ensemble했습니다.

| Model | OOF MAE |
|---|---:|
| LGBM Meta Learner | 8.3807 |
| Huber Regression | 8.4379 |
| **Final Blend** | **8.3773** |

최종 Leaderboard MAE는 **8.5201**을 기록했습니다.

> **Final OOF MAE: 8.3773**  
> **Final Leaderboard MAE: 8.5201**


## 💡 주요 시사점

- **단일 모델보다 Stacking Ensemble에서 더 높은 성능을 확인했습니다.**  
  가장 좋은 Base Model인 `chemprop_bag_all`의 OOF MAE는 8.6065였지만, 여러 Base Model의 예측값을 결합한 최종 Stacking Ensemble은 **OOF MAE 8.3773**까지 개선되었습니다.

- **서로 다른 분자 표현을 사용하는 모델의 예측값을 하나의 Meta Model에서 결합할 수 있었습니다.**  
  최종 모델에는 Chemprop, LightGBM, Morgan Fingerprint, ChemBERTa 등 서로 다른 입력 표현과 모델 구조에서 생성된 예측값이 함께 사용되었습니다.

- **Base Model의 prediction뿐 아니라 물리화학적 특성을 Meta Feature로 함께 사용하는 구조가 활용되었습니다.**  
  10개의 Base Model prediction에 LogP, MolWt, TPSA 등 10개의 물리화학적 Feature를 추가하여 총 20개의 Meta Feature를 구성했습니다.

- **서로 다른 특성을 가진 두 Meta Model을 다시 결합했을 때 가장 낮은 OOF MAE를 기록했습니다.**  
  LGBM L1 Meta Learner의 OOF MAE는 8.3807, Huber Regression은 8.4379였으며, 두 모델을 **7:3 비율로 결합한 최종 모델이 OOF MAE 8.3773**을 기록했습니다.

- 최종적으로 **Leaderboard MAE 8.5201**을 기록했으며, 복수의 분자 표현과 여러 단계의 Ensemble을 활용하는 것이 본 데이터에서 효과적인 접근임을 확인할 수 있었습니다.
