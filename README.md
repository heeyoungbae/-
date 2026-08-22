# Seoul Restaurant Commercial-Area Survival Analysis

서울시 일반음식점 인허가 정보와 서울시 상권분석서비스 데이터를 결합하여  
**음식점의 생존기간과 상권 특성이 폐업 위험에 어떤 관련이 있는지 분석하는 프로젝트**입니다.

주요 분석 방법은 Kaplan–Meier 생존곡선과 Cox Proportional Hazards Model입니다.

## Project Overview

음식점의 개업일과 폐업일을 이용해 생존기간(`duration_days`)과 폐업 여부(`event`)를 정의하고,
음식점 위치를 서울시 상권 공간정보와 결합합니다.

이후 점포 수, 프랜차이즈 비율, 상권 매출, 유동인구, 상주인구 등의 변수를 추가하여
상권 특성과 음식점 폐업 위험의 관계를 분석합니다.

## Analysis Pipeline

1. 음식점 인허가 데이터 전처리
2. 생존기간 및 폐업 이벤트 생성
3. 음식점 업태 분류
4. 음식점 좌표와 서울시 상권 공간정보 Spatial Join
5. 연도별 점포 데이터 결합
6. 연도별 추정매출 데이터 결합
7. 유동인구 및 상주인구 변수 생성
8. 모델링 데이터셋 구축 및 결측치 확인
9. Kaplan–Meier Survival Analysis
10. Cox Proportional Hazards Model
11. Hazard Ratio Forest Plot

## Main Variables

| Category | Variables |
|---|---|
| Survival | `duration_days`, `event` |
| Restaurant | 업태구분명, 소재지면적 |
| Store | 점포 수, 폐업점포 비율, 프랜차이즈 비율 |
| Sales | 점포당 매출금액 |
| Floating population | 총 유동인구, 20·30대 유동인구 비율, 점심/저녁·야간 유동인구 비율 |
| Resident population | 총 상주인구, 20·30대 상주인구 비율 |
| Commercial area | 관광특구 여부 등 |

## Repository Structure

```text
seoul-restaurant-survival-analysis/
├── README.md
├── requirements.txt
├── .gitignore
├── seoul_restaurant_survival_analysis.ipynb
├── data/
│   ├── README.md
│   ├── raw/          # 원본 데이터: GitHub에 업로드하지 않음
│   └── processed/    # 필요한 경우 가공 데이터 저장
└── figures/          # 그래프 저장 폴더
```

## Data

본 저장소에는 원본 데이터를 포함하지 않습니다.

서울시 공공데이터를 직접 다운로드한 뒤 `data/raw/` 폴더에 배치해야 합니다.

노트북에서 사용하는 주요 데이터는 다음과 같습니다.

### Required datasets

- `서울시 일반음식점 인허가 정보.csv`
- 서울시 상권 영역 공간정보(Shapefile)
  - `commercial_area.shp`
  - `commercial_area.shx`
  - `commercial_area.dbf`
  - `commercial_area.prj`
  - `commercial_area.cpg` (있는 경우)
- 서울시 상권분석서비스 **점포-상권** 데이터
  - 2021~2025년 파일
  - 파일명에 `점포-상권`과 해당 연도가 포함되어 있어야 합니다.
- 서울시 상권분석서비스 **추정매출-상권** 데이터
  - 2021~2025년 파일
  - 파일명에 `추정매출-상권`과 해당 연도가 포함되어 있어야 합니다.
- `서울시 상권분석서비스(길단위인구-상권).csv`
- `서울시 상권분석서비스(상주인구-상권).csv`

> 데이터 파일명은 실제 다운로드 시점의 파일명과 다를 수 있습니다.  
> 필요한 경우 노트북의 파일명 또는 검색 조건을 수정하세요.

## Installation

Python 가상환경 사용을 권장합니다.

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

macOS / Linux:

```bash
source .venv/bin/activate
```

필요한 패키지를 설치합니다.

```bash
pip install -r requirements.txt
```

## Run

Jupyter Notebook을 실행합니다.

```bash
jupyter notebook
```

그리고 다음 파일을 순서대로 실행합니다.

```text
seoul_restaurant_survival_analysis.ipynb
```

노트북은 저장소 루트에서 실행하는 것을 기준으로 하며 데이터 경로는 다음과 같이 설정되어 있습니다.

```python
PROJECT_ROOT = Path.cwd()
DATA_DIR = PROJECT_ROOT / "data" / "raw"
```

## Methods

### Kaplan–Meier Estimator

업태 또는 상권 특성에 따른 생존확률 차이를 시각적으로 비교합니다.

### Cox Proportional Hazards Model

상권 변수들이 음식점 폐업 위험에 미치는 영향을 Hazard Ratio로 해석합니다.

- `HR > 1` : 해당 변수가 증가할수록 폐업 위험 증가
- `HR < 1` : 해당 변수가 증가할수록 폐업 위험 감소
- `HR = 1` : 폐업 위험과 뚜렷한 관계 없음

연속형 변수 중 규모 차이가 큰 변수는 `log1p()` 변환 후 모델에 사용합니다.

## Output

분석의 주요 결과는 다음과 같습니다.

- Kaplan–Meier survival curves
- Log-rank test results
- Cox proportional hazards model summary
- Hazard ratio and 95% confidence interval
- Hazard Ratio Forest Plot

## Notes

- CSV 파일은 주로 `EUC-KR` 인코딩으로 불러옵니다.
- Shapefile은 `.shp`뿐 아니라 `.shx`, `.dbf`, `.prj` 등의 구성 파일도 함께 필요합니다.
- 원본 공공데이터는 용량과 재배포 문제를 고려하여 GitHub에서 제외합니다.
- 분석 결과는 관찰자료 기반의 연관성 분석이며 인과관계를 직접 의미하지 않습니다.

## Tech Stack

- Python
- pandas
- NumPy
- GeoPandas
- Matplotlib
- lifelines
- Jupyter Notebook

## License / Data Source

분석 코드는 개인 프로젝트 및 포트폴리오 목적으로 공개할 수 있습니다.

원본 데이터의 이용 및 재배포 조건은 각 서울시 공공데이터의 이용약관을 따르세요.
