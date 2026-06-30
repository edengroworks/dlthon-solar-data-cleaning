# dlthon-solar-data-cleaning
태양광 패널 이미지의 중복·라벨 오류·비관련 데이터를 정제하고, 먼지 여부를 분류하는 데이터 중심 DLthon 프로젝트

# 프로젝트 개요
이번 프로젝트에서는 여러 경로를 통해 수집된 태양광 패널 이미지 데이터를 활용하여 패널의 먼지 오염 여부를 분류합니다.

수집된 데이터에는 동일하거나 매우 유사한 이미지, 잘못 부여된 라벨, 태양광 패널과 관련 없는 이미지가 포함되어 있어 모델 학습 성능을 저하시킬 가능성이 있습니다.
이에 따라 단순히 모델 구조를 변경하는 방식보다, 데이터의 품질을 점검하고 정제하는 **데이터 중심 접근**을 적용합니다.

먼저 EfficientNet-B0 기반 분류 모델을 베이스라인으로 설정하고, 모델과 주요 학습 조건을 고정합니다. 이후 중복 이미지, 라벨 오류 의심 이미지, 비관련 이미지를 탐색하고 정제한 뒤, 정제 전후의 ROC-AUC 성능을 비교합니다.

이를 통해 어떤 데이터 문제가 모델 성능에 영향을 미치는지 분석하고, 태양광 패널 이미지 분류에 적합한 데이터 정제 기준과 학습 파이프라인을 제안하는 것을 목표로 합니다.

# 데이터 구성
- 학습 이미지: 1,366장
- 테스트 이미지 : 468장
- 라벨
  - `0`: Clean
  - `1`: Dusty
- 제출값
  - `dusty_prob`: 테스트 이미지가 Dusty일 확률
    
# 사용 모델
- 모델: EfficientNet-B0
- 사전학습: ImageNet pretrained
- 입력 크기: 224 × 224
- 출력: Dusty 여부를 나타내는 logit 1개
- 분류 헤드
  - Dropout 0.3
  - Linear Layer
- 손실함수: BCEWithLogitsLoss
- Optimizer: AdamW
- 평가 지표: ROC-AUC

# 데이터 중심 접근

다음 세 가지 데이터 문제를 중심으로 정제 실험을 진행합니다.

## 1. 중복 이미지

- 완전히 동일한 이미지
- 크기, 밝기, 회전, 잘림 등이 일부 다른 근접 중복 이미지
- Train과 Validation에 유사 이미지가 동시에 포함될 경우 발생할 수 있는 데이터 누수 확인

## 2. 라벨 오류 의심 이미지

- Clean 이미지에 Dusty 라벨이 부여된 경우
- Dusty 이미지에 Clean 라벨이 부여된 경우
- 모델 예측과 실제 라벨이 크게 다른 이미지 검토

## 3. 비관련 이미지

- 태양광 패널이 없는 이미지
- 제품 그래픽이나 카탈로그 이미지
- 패널보다 주변 풍경이나 다른 물체가 중심인 이미지

# 실험 방법

모델 구조와 주요 학습 조건을 고정하고, 데이터만 변경하여 성능 변화를 비교합니다.

| 실험 | 데이터 조건 | 모델 | 평가 지표 |
|---|---|---|---|
| Baseline | 원본 데이터 | EfficientNet-B0 | ROC-AUC |
| Experiment 1 | 중복 이미지 정제 | EfficientNet-B0 | ROC-AUC |
| Experiment 2 | 비관련 이미지 정제 | EfficientNet-B0 | ROC-AUC |
| Experiment 3 | 라벨 오류 의심 이미지 정제 | EfficientNet-B0 | ROC-AUC |
| Final | 정제 항목 결합 | EfficientNet-B0 | ROC-AUC |

# 프로젝트 평가 기준

- 데이터 EDA와 전처리가 적절하게 이루어졌는가
- 태양광 패널 먼지 분류 Task에 적절한 모델을 선정했는가
- 성능 향상을 위해 논리적으로 데이터 정제 실험을 진행했는가
- 한 번에 하나의 변인을 변경하고 나머지 조건을 통제했는가
- 실험 결과와 실패 원인을 기록했는가
- ROC-AUC를 평가 지표로 사용한 이유를 설명할 수 있는가
- 정제 전후 성능 차이에 대한 결론이 설득력 있는가

# 프로젝트 구조

```text
dlthon-solar-data-cleaning/
├── README.md
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_baseline.ipynb
│   ├── 03_data_cleaning.ipynb
│   └── final_dlthon.ipynb
├── outputs/
│   ├── experiment_results.csv
│   └── submission_final.csv
├── images/
│   └── 실험 및 발표용 이미지
└── presentation/
    └── DLthon_presentation.pptx
