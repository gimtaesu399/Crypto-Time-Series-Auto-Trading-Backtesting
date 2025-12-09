📌 ML-Based Crypto Trading Logic System
Handcrafted Strategy → Backtesting → Probability Modeling → Market Regime Strategies

1. 프로젝트 개요

이 프로젝트는 다음의 전체 파이프라인을 구축하는 것을 목표로 한다:

사용자(트레이더이자 개발자)가 직접 정의한 매매 로직(전략)을 규칙 형태로 구조화한다.

과거 데이터에서 해당 로직을 백테스트하여 성능을 검증한다.

백테스트 결과로부터 "전략이 성공할 구간"을 라벨링하고, 이를 예측하는 머신러닝 모델을 학습한다.

상승장·하락장·횡보장 등의 시장 레짐을 분류한 뒤, 각 레짐에 맞는 전략 및 모델을 적용한다.

최종적으로 예측 기반 자동 매매 또는 모의 매매 환경을 구축한다.

이 시스템은 전략 수립 → 검증 → 모델화 → 시장 상황별 전략 적용의 전체 과정을 자동화하는 구조를 갖는다.

2. 핵심 구성 요소
2.1 전략(Handcrafted Logic)

사용자가 직접 차트를 분석하여 "효과적이라고 판단되는 조건"을 전략으로 정의한다.

예: 이동평균 배열 상태, 거래량 변화, 특정 봉 패턴, 변동성 수축 또는 확장 등.

전략은 Python 함수 또는 JSON 규칙 형태로 저장된다.

2.2 전략 백테스트

특정 전략이 과거 데이터에서 어떤 성능을 보였는지 검증한다.

전략 성공 여부를 기준으로 0/1 라벨을 생성한다.

이 라벨은 이후 모델 학습의 타깃으로 사용된다.

2.3 확률 예측 모델

입력 데이터는 최근 N개의 OHLCV 및 파생지표.

출력은 “해당 전략이 현재 구간에서 성공할 확률”.

초기 모델은 Logistic Regression, Random Forest 등 단순 모델을 사용하며 이후 확장 가능.

2.4 시장 레짐 분류

단순 지표 또는 ML 모델을 통해 시장을 상승장, 하락장, 횡보장으로 분류한다.

각 레짐별로:

다른 전략을 사용할 수 있고

다른 모델을 적용할 수도 있다.

2.5 모의 매매 및 실행 엔진

모델 확률과 전략 조건에 따라 진입/청산/비중 조절을 수행한다.

실시간 또는 히스토리 기반으로 동작.

3. 전체 파이프라인
[1] 전략 정의 (Rule Construction)
      ↓
[2] 백테스트 수행 (Backtesting)
      ↓
[3] 백테스트 결과 기반 라벨 생성 (Success Label)
      ↓
[4] ML 모델 학습 (Probability Model)
      ↓
[5] 시장 레짐 분류 (Regime Detection)
      ↓
[6] 레짐별 전략 및 모델 선택 (Regime-Specific Logic)
      ↓
[7] 모델 기반 자동매매 or 모의매매 실행

4. 프로젝트 구조
project/
 ┣ data/
 ┃ ┣ raw/                 # 원본 OHLCV 데이터
 ┃ ┗ processed/           # 전처리된 데이터
 ┣ strategies/
 ┃ ┣ handcrafted/         # 수동 정의 전략(JSON / py 함수)
 ┃ ┗ regime_based/        # 레짐별 전략 구성
 ┣ models/
 ┃ ┣ base/                # 기본 모델(Logistic, RF 등)
 ┃ ┗ regime/              # 레짐별 모델
 ┣ src/
 ┃ ┣ preprocessing/
 ┃ ┃ ┣ load_data.py
 ┃ ┃ ┗ feature_engineering.py
 ┃ ┣ strategy/
 ┃ ┃ ┣ rule_loader.py     # 전략 규칙 불러오기
 ┃ ┃ ┣ executor.py        # 전략 실행 엔진
 ┃ ┃ ┗ labeler.py         # 성공/실패 라벨 생성
 ┃ ┣ backtest/
 ┃ ┃ ┗ backtester.py      # 백테스트 모듈
 ┃ ┣ model/
 ┃ ┃ ┣ train.py           # 모델 학습
 ┃ ┃ ┗ predict.py         # 확률 예측
 ┃ ┣ regime/
 ┃ ┃ ┗ detector.py        # 시장 상승/하락/횡보 분류
 ┃ ┣ trading/
 ┃ ┃ ┣ simulator.py       # 모의매매 엔진
 ┃ ┃ ┗ position_manager.py
 ┃ ┗ utils/
 ┗ README.md

5. 데이터 구조
5.1 입력 데이터 (OHLCV)
컬럼	의미
timestamp	시간
open	시가
high	고가
low	저가
close	종가
volume	거래량
5.2 모델 입력 Feature

최근 N개의 OHLCV

이동평균, RSI, 볼린저밴드 등 파생지표

변동성 지표(ATR 등)

전략 관련 feature (전략 실행 조건과 직접 연관된 값)

5.3 라벨(Label)
strategy_success(t) = 1 if 전략이 t 시점 진입 후 +X% 도달
                      0 otherwise

6. 모델 구조
6.1 초기 모델

Logistic Regression (baseline)

RandomForestClassifier

6.2 출력
P(t) = 전략 성공 확률 (0~1)

6.3 활용 방식

확률 기반 포지션 크기 조절

확률이 낮으면 진입하지 않음

확률 분포 기반 필터링으로 전략 품질 향상

7. 시장 레짐 분류
단순 기준

상승장: 가격 > MA200

하락장: 가격 < MA200

횡보장: 변동성 축소 구간

고급 기준

클러스터링 기반(HMM 등)

변동성·추세 기반 ML 분류기

레짐 기반 전략 선택
if regime == UP:    uptrend_strategy + uptrend_model
if regime == DOWN:  downtrend_strategy + downtrend_model
if regime == SIDE:  sideway_strategy + side_model

8. 매매 실행
진입 규칙 예시
P(t) ≥ 0.7 → 매수
0.6 ≤ P(t) < 0.7 → 소액 매수
P(t) < 0.6 → 진입하지 않음

비중 조절
position_size = max(0, (P(t) - 0.5) * 2)

청산 규칙

확률 하락 시 청산

손절/익절 조건 충족 시 청산

레짐 변화 시 전략 변경

9. 개발 단계 로드맵
Phase 1 — 전략 규칙 정의

사용자 직관 기반 로직 정리

JSON 또는 Python 함수로 저장

Phase 2 — 백테스트

과거 데이터로 전략 검증

성공/실패 라벨 생성

Phase 3 — 모델 학습

기본 모델로 시작하여 확률 예측

Feature 튜닝 및 성능 개선

Phase 4 — 확률 기반 전략 + 백테스트

기존 전략 대비 성능 비교

Phase 5 — 시장 레짐 추가

레짐 분류 알고리즘 구축

레짐별 로직/모델 구성

Phase 6 — 모의매매/실시간 시스템 구축

자동매매 엔진 완성
