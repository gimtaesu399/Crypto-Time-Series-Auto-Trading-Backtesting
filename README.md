ML 기반 암호화폐 트레이딩 로직 시스템
Handcrafted Strategy → Backtesting → Probability Modeling → Market Regime Strategies
📌 프로젝트 개요

본 프로젝트는 사람이 직접 설계한 매매 전략을 기반으로,
이를 백테스트 → 확률 모델링 → 시장 상황별 전략 선택으로 확장하는
ML 기반 트레이딩 파이프라인을 구축하는 것을 목표로 한다.

핵심 철학은 다음과 같다.

“트레이더의 직관으로 만든 전략을
과거 데이터로 검증하고,
머신러닝으로 ‘성공할 확률’을 정량화하여
시장 상황에 맞게 선택·실행한다.”

즉, 단순한 자동매매가 아니라
전략 수립 → 검증 → 모델화 → 레짐별 적용의 전 과정을 자동화한 시스템이다.

🔄 전체 파이프라인
[1] 전략 정의 (Rule Construction)
        ↓
[2] 전략 백테스트 (Backtesting)
        ↓
[3] 백테스트 결과 기반 라벨 생성 (Success Labeling)
        ↓
[4] 머신러닝 모델 학습 (Probability Modeling)
        ↓
[5] 시장 레짐 분류 (Regime Detection)
        ↓
[6] 레짐별 전략 및 모델 선택
        ↓
[7] 확률 기반 모의매매 / 자동매매 실행

🧩 핵심 구성 요소
2.1 전략 정의 (Handcrafted Strategy)

사용자가 차트를 직접 분석하여
효과적이라고 판단되는 매매 조건을 규칙 형태로 정의

예시:

이동평균 배열 상태

거래량 급증 / 감소

특정 캔들 패턴

변동성 수축 또는 확장 구간

전략 저장 방식:

Python 함수

JSON 기반 규칙 파일

📌 목적

인간의 트레이딩 직관을 재현 가능하고 검증 가능한 형태로 구조화

2.2 전략 백테스트

정의된 전략을 과거 OHLCV 데이터에 적용

각 시점에서:

전략 진입 조건 충족 여부 판단

진입 후 일정 조건(+X%, 손절/익절 등) 도달 여부 확인

결과를 기반으로 성공/실패 라벨 생성

strategy_success(t) = 
    1  (전략 성공)
    0  (전략 실패)


📌 이 라벨은 이후 ML 모델 학습의 정답(label) 으로 사용된다.

2.3 확률 예측 모델 (Probability Model)

입력 데이터

최근 N개의 OHLCV

이동평균, RSI, 볼린저 밴드 등 파생 지표

ATR 등 변동성 지표

전략 조건과 직접 연관된 feature

출력

현재 시점에서 전략이 성공할 확률

𝑃
(
𝑡
)
∈
[
0
,
1
]
P(t)∈[0,1]
초기 적용 모델

Logistic Regression (baseline)

Random Forest Classifier

📌 목적

“이 전략이 지금 시점에서 얼마나 성공할 가능성이 있는가”를 수치화

2.4 시장 레짐 분류 (Market Regime Detection)

시장을 다음과 같은 레짐으로 구분한다.

상승장 (Uptrend)

하락장 (Downtrend)

횡보장 (Sideways)

단순 기준

상승장: 가격 > MA200

하락장: 가격 < MA200

횡보장: 변동성 축소 구간

고급 확장 (선택)

변동성·추세 기반 클러스터링

Hidden Markov Model (HMM)

ML 기반 레짐 분류기

2.5 레짐 기반 전략 및 모델 선택

시장 레짐에 따라 서로 다른 전략과 모델을 적용한다.

if regime == "UP":
    use uptrend_strategy + uptrend_model
elif regime == "DOWN":
    use downtrend_strategy + downtrend_model
elif regime == "SIDE":
    use sideways_strategy + side_model


📌 목적

모든 시장에 하나의 전략을 쓰지 않고,
상황에 맞는 전략을 선택

2.6 모의매매 및 실행 엔진
진입 규칙 예시

𝑃
(
𝑡
)
≥
0.7
P(t)≥0.7 → 적극 진입

0.6
≤
𝑃
(
𝑡
)
<
0.7
0.6≤P(t)<0.7 → 소액 진입

𝑃
(
𝑡
)
<
0.6
P(t)<0.6 → 진입하지 않음

비중 조절
𝑝
𝑜
𝑠
𝑖
𝑡
𝑖
𝑜
𝑛
_
𝑠
𝑖
𝑧
𝑒
=
max
⁡
(
0
,
(
𝑃
(
𝑡
)
−
0.5
)
×
2
)
position_size=max(0,(P(t)−0.5)×2)
청산 조건

확률 하락 시

손절 / 익절 조건 충족 시

시장 레짐 변경 시

📁 프로젝트 구조
project/
├─ data/
│  ├─ raw/                # 원본 OHLCV 데이터
│  └─ processed/          # 전처리된 데이터
│
├─ strategies/
│  ├─ handcrafted/        # 수동 정의 전략 (JSON / Python)
│  └─ regime_based/       # 시장 레짐별 전략
│
├─ models/
│  ├─ base/               # 기본 모델 (Logistic Regression, Random Forest)
│  └─ regime/             # 레짐별 모델
│
├─ src/
│  ├─ preprocessing/
│  │  ├─ load_data.py     # 데이터 로드
│  │  └─ feature_engineering.py  # Feature 생성
│  │
│  ├─ strategy/
│  │  ├─ rule_loader.py   # 전략 규칙 로딩
│  │  ├─ executor.py      # 전략 실행 엔진
│  │  └─ labeler.py       # 성공/실패 라벨 생성
│  │
│  ├─ backtest/
│  │  └─ backtester.py    # 전략 백테스트
│  │
│  ├─ model/
│  │  ├─ train.py         # 확률 모델 학습
│  │  └─ predict.py       # 성공 확률 예측
│  │
│  ├─ regime/
│  │  └─ detector.py      # 시장 레짐 분류
│  │
│  ├─ trading/
│  │  ├─ simulator.py    # 모의매매 엔진
│  │  └─ position_manager.py  # 포지션/비중 관리
│  │
│  └─ utils/              # 공통 유틸리티
│
└─ README.md

🧭 개발 로드맵
Phase 1 — 전략 규칙 정의

사용자 직관 기반 매매 로직 정리

JSON / Python 함수로 저장

Phase 2 — 백테스트

과거 데이터로 전략 검증

성공/실패 라벨 생성

Phase 3 — 모델 학습

기본 분류 모델로 확률 예측

Feature 튜닝

Phase 4 — 확률 기반 전략 평가

기존 전략 대비 성능 비교

Phase 5 — 시장 레짐 추가

레짐 분류 알고리즘 구축

레짐별 전략/모델 분리

Phase 6 — 모의매매 / 실시간 시스템

확률 기반 자동매매 엔진 완성

✨ 프로젝트 의의

인간 트레이더의 전략을 검증 가능한 형태로 구조화

단순 자동매매가 아닌 확률 기반 의사결정 시스템

시장 상황에 따른 동적 전략 선택 구조

연구·실험·확장이 가능한 모듈형 설계
