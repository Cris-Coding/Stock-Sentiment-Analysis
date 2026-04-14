# Stock Sentiment Analysis

A Python tool that fetches financial news articles and measures whether sentiment correlates with real stock price movement. Two sentiment models are compared head-to-head: VADER, a general-purpose NLP analyzer, and AlphaVantage's finance-specific sentiment scorer.

Built for CSI 5810 (Data Science) at Oakland University.

---

## Overview

Financial news influences markets, but not all sentiment tools interpret financial language the same way. A general-purpose model trained on social media text reads the same article very differently than one trained exclusively on financial reporting. This project measures that gap concretely by:

1. Fetching financial news articles for a given ticker via the AlphaVantage API
2. Scoring each article independently with VADER and AlphaVantage's built-in scorer
3. Aggregating daily sentiment scores and merging them with closing price data from `yfinance`
4. Computing Pearson correlation between sentiment and price movement
5. Visualizing both models side-by-side in an interactive desktop GUI

---

## Key Results

Testing was conducted on NVIDIA (NVDA) over a 1-month window using 1,000 articles.

| Model | Pearson Correlation (vs. NVDA close price) |
|---|---|
| VADER | -0.8499 |
| AlphaVantage | -0.7563 |

Both models found a strong negative correlation during this period. VADER showed a consistent positive bias — it overestimated sentiment on financial language that the market interpreted as neutral or bearish. AlphaVantage's domain-specific model tracked actual price movement more closely.

In a direct comparison across 50 articles, only 3 scored nearly identically between the two models. The largest single-article disagreement was 0.945 points on a normalized -1 to 1 scale.

---

## Project Structure

```
stock-sentiment-analysis/
│
├── AV_Sentiment_Analysis.py    # GUI app: AlphaVantage model vs. stock price
├── VADER_Sentiment_Analysis.py # GUI app: VADER model vs. stock price
├── AV_VADER_COMP.py            # Head-to-head scatter plot comparison
├── Sentiment_Test.py           # Early exploratory script
├── article_keys.py             # Dev utility: inspect AlphaVantage article structure
└── CSI 5810 Project 2.pdf      # Full project report
```

---

## Tech Stack

- **Python 3.x**
- [NLTK / VADER](https://github.com/cjhutto/vaderSentiment) — lexicon-based sentiment scoring
- [yfinance](https://github.com/ranaroussi/yfinance) — historical stock price data
- [AlphaVantage News Sentiment API](https://www.alphavantage.co/) — financial news feed and proprietary sentiment scores
- `pandas` — data aggregation and time series merging
- `matplotlib` — dual-axis visualization
- `tkinter` — desktop GUI

---

## Installation

```bash
git clone https://github.com/Cris-Coding/stock-sentiment-analysis.git
cd stock-sentiment-analysis
pip install yfinance pandas matplotlib nltk requests
```

You will need a free [AlphaVantage API key](https://www.alphavantage.co/support/#api-key). Set it as an environment variable rather than hardcoding it:

```bash
export AV_API_KEY=your_key_here
```

---

## Usage

### Interactive GUI — AlphaVantage Model

```bash
python AV_Sentiment_Analysis.py
```

### Interactive GUI — VADER Model

```bash
python VADER_Sentiment_Analysis.py
```

Both GUIs accept the same inputs:

- **Ticker** — any valid stock symbol (NVDA, MSFT, AAPL, etc.)
- **Months** — lookback window, 1 to 24
- **Limit** — number of articles to fetch from AlphaVantage

Output is a dual-axis time series chart overlaying closing price and daily average sentiment score, with the Pearson correlation coefficient displayed above.

### Model Comparison — Scatter Plot

```bash
python AV_VADER_COMP.py
```

Scores articles with both models and plots VADER score vs. AlphaVantage score per article, with a perfect-agreement reference line.

### API Structure Inspector

```bash
python article_keys.py --ticker MSFT
```

Prints all available keys and a formatted preview of each field in an AlphaVantage article response. Useful for development.

---

## Design Decisions

**Missing sentiment days filled with 0**
On days where no articles are available, sentiment is assigned a neutral score of 0 rather than being dropped from the dataset. This preserves the full time series and avoids survivorship bias toward periods with high news coverage.

**Daily sentiment aggregated by mean**
Multiple articles often publish on the same trading day. Since stock data provides one closing price per day, article scores are averaged to maintain a 1:1 mapping for correlation.

**Headline + summary concatenated before scoring**
Headlines alone are frequently ambiguous or sensationalized. Including the article summary in the scoring input produced more stable and contextually accurate results from both models.

**Ticker-specific AV scores preferred over overall scores**
AlphaVantage returns both an overall article sentiment and a per-ticker sentiment score. The per-ticker score is used when available, falling back to the overall score only when the target ticker is not explicitly listed in the article's sentiment data.

---

## Limitations

- AlphaVantage's free tier caps article retrieval per request, limiting the analysis window and reducing sample size for correlation
- VADER was trained on social media text and carries a positive lexical bias that does not translate well to financial language
- Correlation does not imply causation; sentiment may lag price movement rather than lead it
- Analysis uses closing price only; intraday data could surface more nuanced relationships

---

## Potential Extensions

- Lagged correlation analysis — does today's sentiment predict tomorrow's price?
- FinBERT integration — a transformer model fine-tuned specifically on financial text
- Multi-ticker comparison and sector-level sentiment aggregation
- CSV export for downstream statistical analysis

---

## Report

Full methodology, figures, and conclusions: `CSI 5810 Project 2.pdf`
