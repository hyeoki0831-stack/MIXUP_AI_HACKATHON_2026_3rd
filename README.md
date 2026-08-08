# MIXUP_AI_HACKATHON_2026_3rd
MIXUP AI HACKATHON 2026 3rd Track 2

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

**데이터 설명**

본 대회에서는 각 화합물의 분자 구조를 나타내는 **SMILES**와,
SMILES로부터 추출한 여러 형태의 분자 feature가 제공됩니다.

- **Train:** 39,994개 화합물
- **Test:** 9,998개 화합물
- **Target:** `hERG_inhibition`
- `hERG_inhibition`은 **1µM 농도에서 측정된 hERG percent inhibition 값**입니다.

Train 데이터에는 정답인 `hERG_inhibition`이 포함되어 있으며,
Test 데이터에는 정답이 제공되지 않습니다.

### 주요 변수 설명

| 변수 그룹 | 주요 변수명 | 설명 |
|---|---|---|
| **식별자** | `id` | 각 화합물 샘플을 구분하기 위한 고유 ID |
| **분자 구조** | `smiles` | 원자와 결합으로 이루어진 화합물의 분자 구조를 문자열 형태로 표현 |
| **타깃 변수** | `hERG_inhibition` | 1µM 농도에서 측정된 hERG 채널 억제율로, 모델이 예측해야 하는 연속형 변수 |
| **전자적 특성·표면적** | `EState_VSA4`, `PEOE_VSA8`, `EState_VSA8` 등 | 원자의 전자적 상태 또는 부분전하와 분자 표면적 정보를 함께 나타내는 RDKit Descriptor |
| **분자 연결성** | `Chi1n`, `Chi2n`, `Chi3n`, `Chi4n` 등 | 분자 내 원자들의 연결 관계와 구조적 복잡성을 수치화한 Descriptor |
| **분자 구조·형태** | `BalabanJ`, `Kappa2` 등 | 분자의 위상 구조와 형태적 복잡성을 나타내는 Descriptor |
| **물리화학적 특성** | `MolMR`, `SMR_VSA6`, `SlogP_VSA8` 등 | 분자의 굴절률, 지용성 및 이와 관련된 표면적 특성을 나타내는 변수 |
| **방향족 구조** | `NumAromaticRings`, `NumAromaticHeterocycles` 등 | 분자 내 방향족 고리와 방향족 헤테로고리의 개수를 나타내는 구조 변수 |
| **부분구조 패턴** | Morgan Fingerprint 2,048 bit | 원자 주변의 국소적인 분자 구조 패턴을 0/1 형태의 고차원 벡터로 표현 |
| **SMILES 잠재 표현** | ChemBERTa Embedding 768차원 | Transformer가 SMILES 문자열에서 학습한 분자의 고차원 잠재 표현 |

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
