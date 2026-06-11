# 📈 StockLens — Offline Visual Stock Analyser

> A fully offline, browser-based stock analysis tool. No API keys. No backend. No internet required after the page loads. Paste your CSV data and get instant charts, technical signals, a 3-day price forecast, and a recovery roadmap.

---

## 🖥️ Live Demo

Open `StockLens.html` in any browser. No installation. No server. No dependencies to install.

---

## ✨ Features

| Feature | Description |
|---|---|
| 📊 Price chart | 6-month price history with 20-day and 50-day moving average overlays |
| 🎯 Signal matrix | 4 technical signals scored and visualised as progress bars |
| 📍 Range bar | Visual pin showing where price sits in its 52-week range |
| 📅 3-day forecast | Day-by-day directional prediction with expected price ranges |
| 📊 Momentum chart | 30-session bar chart showing % gain/loss vs period start |
| 🗺️ Recovery roadmap | Timeline of when and why the stock is likely to rise again |
| 👁️ Watch list | 4 specific levels and signals to monitor |
| ✅ Verdict | One-paragraph bottom line with colour-coded signal strength |

---

## 🚀 How to Use

### Step 1 — Get your stock data
Go to one of these free sources and download historical price data:

- **Yahoo Finance**: `finance.yahoo.com` → search ticker (e.g. `TATASTEEL.NS`) → Historical Data → Download CSV
- **NSE India**: `nseindia.com` → search stock → Historical Data
- **Moneycontrol / Investing.com**: Export price history as CSV

### Step 2 — Paste the CSV
Open the downloaded file in Notepad / TextEdit, select all (`Ctrl+A`), copy, and paste into the text area in the app.

**Supported formats:**

```
# Yahoo Finance format (auto-detected)
Date,Open,High,Low,Close,Adj Close,Volume
2025-01-02,148.30,152.10,146.80,150.40,150.40,4823000
2025-01-03,150.40,155.20,149.10,153.80,153.80,5201000

# Minimal format (just two columns)
Date,Close
01-Jun-2025,206.77
02-Jun-2025,204.30
```

The parser automatically finds the `Date` and `Close` columns regardless of column order or extra columns. Minimum **30 rows** recommended for accurate signals; 120+ rows gives the best results.

### Step 3 — Hit Analyse
All computation runs in your browser. Results appear instantly.

---

## 🧮 Technical Methodology — Full Formula Reference

This section documents every formula and signal used in the analysis engine.

---

### 1. Moving Averages (MA)

**Simple Moving Average (SMA):**

```
MA(n) = (Close[0] + Close[1] + ... + Close[n-1]) / n
```

Where `n` is the period and `Close[0]` is the most recent close.

Two MAs are computed:
- **20-day MA** — short-term trend indicator. Price above = bullish. Price below = bearish.
- **50-day MA** — medium-term trend indicator. Confirms or contradicts the 20-day signal.

MA lines are plotted on the price chart for visual reference.

---

### 2. Relative Strength Index (RSI)

**RSI Formula (Wilder's, 14-period):**

```
RS  = Average Gain (14 periods) / Average Loss (14 periods)
RSI = 100 - (100 / (1 + RS))
```

Where:
- Average Gain = mean of all up-closes over last 14 sessions
- Average Loss = mean of all down-closes (absolute value) over last 14 sessions

**Interpretation used:**

| RSI Range | Signal | Action |
|---|---|---|
| < 35 | Oversold | +1 bullish point — bounce likely |
| 35 – 68 | Neutral | No contribution to score |
| > 68 | Overbought | −1 bearish point — pullback likely |

---

### 3. Price Momentum

**5-day momentum:**
```
Mom5 = ((Close[today] - Close[5 days ago]) / Close[5 days ago]) × 100
```

**20-day momentum:**
```
Mom20 = ((Close[today] - Close[20 days ago]) / Close[20 days ago]) × 100
```

**Scoring:**
```
if Mom5  >  +2%  →  momScore + 1
if Mom5  <  -2%  →  momScore - 1
if Mom20 >  +6%  →  momScore + 1
if Mom20 <  -6%  →  momScore - 1
```

---

### 4. Support & Resistance

```
Support    = min(Low[last 20 sessions])  × 0.998
Resistance = max(High[last 20 sessions]) × 1.002
```

The 0.2% buffer prevents false breakouts from touching the exact level.

---

### 5. 52-Week Range Position

```
RangePos = ((Current Price - Period Low) / (Period High - Period Low)) × 100
```

Expressed as a percentage (0% = at low, 100% = at high).

**Scoring:**
```
RangePos > 72%  →  rangeScore - 1  (stretched, mean-reversion risk)
RangePos < 28%  →  rangeScore + 1  (value zone, upside potential)
28% – 72%       →  rangeScore = 0  (neutral)
```

---

### 6. Signal Scoring Matrix

All four signals are scored and summed into an **Overall Score**:

```
overallScore = techScore + momentumScore + dayScore + rangeScore
```

| Signal | Max contribution | How it's scored |
|---|---|---|
| Technical (MA + RSI) | −3 to +3 | See sections 1 & 2 |
| Momentum (5d + 20d) | −2 to +2 | See section 3 |
| Today's session | −1 to +1 | `chgPct > +0.7%` = +1, `< -0.7%` = -1 |
| Range position | −1 to +1 | See section 5 |

**Overall verdict mapping:**

| Score | Verdict | Colour |
|---|---|---|
| ≥ +3 | STRONG BUY | Green |
| +2 | BULLISH | Green |
| +1 | LEAN UP | Light green |
| 0 | NEUTRAL | Amber |
| −1 | LEAN DOWN | Light red |
| ≤ −2 | BEARISH | Red |

---

### 7. 3-Day Forecast Logic

The forecast is scenario-based, not regression-based. The `overallScore` and `RSI` together select one of 6 scenario sets:

| Condition | Scenario set |
|---|---|
| RSI < 36 AND score ≥ −1 | Oversold bounce: FLAT → UP → UP |
| score ≥ +2 | Strong bull: UP → UP → FLAT |
| score ≤ −2 | Strong bear: DOWN → DOWN → FLAT |
| score = +1 | Mild bull: UP → UP → FLAT |
| score = −1 | Mild bear: DOWN → FLAT → FLAT |
| score = 0 | Neutral: FLAT → FLAT → FLAT |

**Price range per day:**
```
Day N Low  = Current Price × rangeLow[N]
Day N High = Current Price × rangeHigh[N]
```

Range multipliers are tighter for flat days (±0.5–1%) and wider for strong directional days (±1.5–2.5%).

---

### 8. Momentum Chart

For each of the last 30 sessions:
```
MomentumBar[i] = ((Close[i] - Close[0]) / Close[0]) × 100
```

Where `Close[0]` is the close 30 sessions ago. Bars are green if positive, red if negative.

---

### 9. Recovery Roadmap

The roadmap is generated conditionally based on `overallScore`:

**Bearish scenario (score ≤ −1):**
- Stage 1: Current selling / sideways pressure
- Stage 2: Support test and base-building (1–2 weeks)
- Stage 3: Recovery to 20-day MA or resistance (2–4 weeks)
- Stage 4: Full recovery to 50-day MA or prior high (4–8 weeks)

**Bullish scenario (score ≥ 0):**
- Stage 1: Current supportive trend
- Stage 2: Next resistance level
- Stage 3: Medium-term momentum target

Recovery targets:
```
Target 1 = price < MA20  ?  MA20  :  Resistance
Target 2 = price < MA50  ?  MA50  :  Period High × 0.95
```

---

## 📁 Project Structure

```
StockLens/
├── StockLens.html    ← Entire app (HTML + CSS + JS, single file)
└── README.md         ← This file
```

The entire application is a single HTML file (~800 lines). No build step, no package manager, no framework.

**Dependencies (loaded from CDN, only needed once):**
- [Chart.js 4.4.1](https://www.chartjs.org/) — charting library
- [Google Fonts](https://fonts.google.com/) — Inter + IBM Plex Mono

After the first load, the app works fully offline.

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| UI | Vanilla HTML5 + CSS3 (CSS custom properties, Grid, Flexbox) |
| Charts | Chart.js 4.4.1 |
| Analysis engine | Vanilla JavaScript (ES6+) |
| Data input | CSV parsing (regex-based, handles multiple formats) |
| Fonts | Inter (UI) + IBM Plex Mono (numbers/labels) |
| Backend | None |
| Database | None |
| API | None |

---

## ⚠️ Limitations & Disclaimer

- **Technical analysis only** — the app does not analyse news, fundamentals, earnings, dividends, or macro events. For complete analysis, combine with news research.
- **No real-time data** — you must paste historical data manually. The analysis reflects the data you provide.
- **Forecast is scenario-based** — not a machine learning prediction. It uses rule-based pattern matching from established technical analysis frameworks.
- **Past patterns do not guarantee future outcomes** — no technical indicator is 100% reliable.

> ⚠️ **This tool is not financial advice.** Always consult a SEBI-registered investment advisor (India) or licensed financial professional before making investment decisions.

---

## 📜 License

MIT License — free to use, modify, and distribute.

---

## 🙏 Credits

Built with [Chart.js](https://www.chartjs.org/) · Fonts by [Google Fonts](https://fonts.google.com/)

Analysis methodology based on standard technical analysis principles: Wilder's RSI (J. Welles Wilder, 1978), Simple Moving Averages, and price momentum indicators.
