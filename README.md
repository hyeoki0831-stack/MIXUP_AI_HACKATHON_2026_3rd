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
- 대회 링크:https://dacon.io/competitions/official/236720/overview/description
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

SMILES로부터 계산한 **분자의 물리·화학적 및 구조적 특성**입니다.

총 **210개의 feature**가 제공되며, 대표적으로 다음과 같은 정보를 포함합니다.

- 분자량(Molecular Weight)
- LogP
- TPSA
- 수소결합 Donor / Acceptor
- Rotatable Bond
- Ring 관련 특성

즉, Descriptor는

> **"이 분자가 어떤 물리·화학적 특성을 가지고 있는가?"**

를 수치로 표현한 데이터입니다.

#### 2. Morgan Fingerprint

분자 내부의 **국소적인 구조 및 부분구조 패턴**을 벡터 형태로 표현한 데이터입니다.

- Radius: 2
- Dimension: **2,048**
- 값: **0 / 1**

예를 들어 특정 원자 주변에 특정 구조 패턴이 존재하는지를 고정된 길이의 벡터로 표현합니다.

즉, Fingerprint는

> **"이 분자가 어떤 구조적 패턴을 가지고 있는가?"**

를 표현하는 데이터입니다.

#### 3. ChemBERTa Feature

SMILES 문자열을 분자 데이터로 사전학습된 Transformer 모델인
**ChemBERTa**를 이용해 숫자 벡터로 변환한 데이터입니다.

- Dimension: **768**

Descriptor처럼 각 feature가 '분자량', '극성'처럼 사람이 정의한 의미를 가지는 것은 아니며,
Transformer가 SMILES에서 학습한 **잠재 표현(Latent Representation)** 입니다.

즉,

> **"SMILES에 포함된 복잡한 분자 구조 정보를 모델이 학습한 표현"**

이라고 볼 수 있습니다.

### 전체 Feature 구성

| Feature 종류 | 차원 |
|---|---:|
| RDKit Descriptor | 210 |
| Morgan Fingerprint | 2,048 |
| ChemBERTa Embedding | 768 |
| **전체** | **3,026** |

`all_feature` 데이터는 위 세 가지 feature를 하나로 결합한 데이터입니다.
