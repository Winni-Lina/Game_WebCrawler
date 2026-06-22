# Steam 리뷰 분석 미니 프로젝트

웹크롤러로 수집한 Steam 리뷰를 기반으로 한 두 가지 미니 프로젝트입니다.

1. **단어 빈도수 분석 대시보드** (`SteamReviewDashboard.py`)
2. **리뷰 감성(긍정/부정) 분류 대시보드** (`SteamSentimentDashboard.py`) — 머신러닝 모델 사용

## 폴더 구조
```
review/
├── data/
│   ├── steam_reviews.csv          # 단어빈도용 (preprocess_steam_reviews.py 생성)
│   └── steam_sentiment.csv        # 감성학습용 review,label (preprocess_sentiment_data.py 생성)
├── model/
│   ├── sa_steam_vectorizer.pkl    # TF-IDF 벡터라이저
│   └── sa_steam_predict.pkl       # 학습된 분류 모델
├── mylib/
│   ├── myTextAnalyzer.py          # 토큰화/빈도 분석
│   ├── myStreamlitVisualizer.py   # 막대그래프/워드클라우드
│   └── sentiment_analyzer.py      # 감성 분석 토크나이저 + 배포 클래스
├── preprocess_steam_reviews.py    # (단어빈도) 크롤러 JSONL → CSV
├── preprocess_sentiment_data.py   # (감성) jsonldata 대용량 JSONL → 균형 학습 CSV
├── train_sentiment_model.py       # 감성 분류 모델 학습/평가/저장
├── SteamReviewDashboard.py        # 단어빈도 Streamlit 앱
├── SteamSentimentDashboard.py     # 감성 분류 Streamlit 앱
├── requirements.txt
└── README.md
```

## 설치
```cmd
"C:\Users\user\anaconda3\python.exe" -m pip install -r requirements.txt
```
> 한글 형태소 분석(`konlpy` Okt)은 Java(JDK)가 필요합니다. (이 PC는 설치 확인됨)

---

## A. 감성(긍정/부정) 분류 — 머신러닝

데이터 소스: `C:\Users\user\Desktop\WebCrawling\jsonldata\*.jsonl`
(각 줄: `{"game","genre","lang","review","voted_up"}`, voted_up = 추천/비추천 라벨)

### 1) 학습 데이터 전처리
```cmd
cd /d C:\Users\user\Desktop\WebCrawling\review
"C:\Users\user\anaconda3\python.exe" preprocess_sentiment_data.py
```
- 대용량 JSONL(약 260만 건)을 스트리밍하며 정제(BBCode/URL 제거, 한글·길이 필터)
- voted_up → label (1=긍정 / 0=부정)
- **클래스 균형 샘플링**(긍정/부정 동수) → `data/steam_sentiment.csv`

### 2) 모델 학습
```cmd
"C:\Users\user\anaconda3\python.exe" train_sentiment_model.py
```
- Okt 토큰화 + TF-IDF(max 3000) 특징 추출
- NaiveBayes / LogisticRegression / LinearSVC 학습·비교 후 **최고 모델 저장**
- 결과 예시: LogisticRegression 테스트 정확도 ≈ **78%**
- `model/sa_steam_vectorizer.pkl`, `model/sa_steam_predict.pkl` 저장
> ⚠️ Okt 형태소 분석이 느려 4만 건 학습에 수십 분 걸립니다(1회성).

### 3) 분류 대시보드 실행
```cmd
"C:\Users\user\anaconda3\python.exe" -m streamlit run SteamSentimentDashboard.py
```
- 단일 리뷰 입력 → 긍정/부정 + 신뢰도
- CSV 일괄 분석 → 긍부정 분포, 결과 다운로드

---

## B. 단어 빈도수 분석 대시보드

```cmd
REM 1) 전처리 (크롤러 output/reviews 의 JSONL → CSV)
"C:\Users\user\anaconda3\python.exe" preprocess_steam_reviews.py

REM 2) 대시보드 실행
"C:\Users\user\anaconda3\python.exe" -m streamlit run SteamReviewDashboard.py
```
- 주제/게임/평점 필터 → 한국어 형태소 분석 → 단어 빈도
- 수평 막대그래프 / 워드클라우드 / 원본 리뷰 테이블

> 이 대시보드의 `data/steam_reviews.csv`는 기존 크롤러 `output/reviews/*.jsonl`을 사용합니다.
> 데이터를 새로 모았다면 `preprocess_steam_reviews.py`를 다시 실행하세요.
