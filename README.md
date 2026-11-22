# Forecast vs. Actual: Flagging Revenue Surprises with Public SEC Data

## Why I built this
At month-end close I compare actual results with forecasts across subsidiaries, but I usually see a problem only after it shows up in the numbers. I wanted to test, with public data, how well simple forecasts anticipate quarterly revenue, which quarters a planner would have flagged for investigation, and how much of the gap a Japanese parent company would see as currency rather than business performance.

## Data
- **Quarterly revenue** for Coca-Cola, UPS and Delta Air Lines (all Atlanta-headquartered, December year end) from the SEC's XBRL company-facts API, Q1 2012 to Q3 2026. Coca-Cola's series starts in Q1 2018 because its Q4 2017 revenue could not be derived from the filings.
- **USD/JPY monthly average rates** from FRED (series `EXJPUS`).

## Method
1. **Clean quarterly series.** Pulled quarterly revenue from 10-Q and 10-K filings, handled the XBRL revenue tags that changed over the years, and derived Q4 as full year minus Q1–Q3.
2. **Rolling forecasts.** Before each of the latest 12 quarters (Q4 2023 – Q3 2026), forecast it using only earlier data, with three models: same quarter last year, last year plus recent growth, and Holt-Winters exponential smoothing. Compared accuracy with MAPE (mean absolute percentage error).
3. **Variance flags.** Flagged quarters where the actual result missed the best model's forecast by more than 5%, then read the companies' earnings releases to find out why.
4. **Yen view.** Treated Coca-Cola as if it were the U.S. subsidiary of a Japanese parent: translated the plan into yen at a budget rate (prior-year Q4 average) and the actual result at the actual rate, then split the gap into operating variance and FX variance.

## Results

### 1. Holt-Winters was the most accurate model for all three companies

| Company | Same quarter last year | Last year + growth | Holt-Winters | Quarters flagged (>5%) |
|---|---|---|---|---|
| Coca-Cola | 4.5% | 3.0% | **2.2%** | 1 |
| UPS | 4.0% | 4.8% | **2.8%** | 2 |
| Delta Air Lines | 7.0% | 6.5% | **4.3%** | 5 |

*MAPE over the 12 test quarters; lower is better.*

Holt-Winters updates its estimates of both the trend and the seasonal pattern every quarter, so it adjusts when growth speeds up or slows down. The two simple methods carry last year's pattern forward. Delta was the hardest to forecast with every model, likely because airline revenue swings more with travel demand and capacity.

### 2. The misses had very different causes
I read the earnings releases for three of the eight flagged quarters:

| Quarter | Company | Actual vs. forecast | What happened | Could a planner have seen it? |
|---|---|---|---|---|
| Q4 2023 | UPS | +8.3% | Customers had moved volume to other carriers during the 2023 labor negotiations; part of it came back after the new contract. | Partly: the dip was temporary, but the model read it as a trend. |
| Q3 2024 | Delta | −8.8% | The July 2024 IT outage cost about USD 380 million in revenue, but that explains only about a quarter of the gap. | No, for the outage itself. |
| Q1 2025 | Coca-Cola | −6.4% | Currency headwinds and the refranchising of bottling operations lowered reported revenue, while organic revenue still grew 6%. | Mostly yes: currency trends and refranchising plans were known in advance. |

A model built only on past revenue cannot tell these cases apart. The forecast is a starting point; the explanation comes from the variance review.

### 3. In yen, exchange rates often outweighed the business itself
With the prior-year Q4 average rate as the budget rate, FX variance was larger than operating variance in 8 of the 12 quarters. In 4 quarters, FX reversed the direction of the yen result:

| Quarter | Operating variance (¥ bn) | Total yen variance (¥ bn) |
|---|---|---|
| Q4 2023 | −62.3 | +5.9 |
| Q2 2025 | +27.2 | −72.8 |
| Q3 2025 | +18.3 | −43.4 |
| Q2 2026 | −5.6 | +64.1 |

In Q4 2023, for example, revenue fell short of plan, but a weaker yen (147.8 vs. a budget rate of 141.5 per dollar) turned the yen result into a small beat. For a U.S. subsidiary, this means performance should be judged at the budget rate, with the FX effect reported separately, and the annual plan should show how sensitive the yen result is to the exchange rate.

## What I learned
Two results surprised me. First, in the yen view, exchange rates mattered more than the business itself in most quarters, so judging a U.S. subsidiary on its yen results alone would often credit or blame it for something it does not control. Second, even a well-known event explained only part of a miss: Delta's outage accounted for about a quarter of its Q3 2024 gap. If I did this again, I would add business drivers such as volumes and pricing, use each company's own constant-currency figures, and investigate all eight flagged quarters instead of three.

## Limitations and next steps
- The yen view is hypothetical: Coca-Cola is not owned by a Japanese company. I used it to show how a parent would see a U.S. subsidiary.
- Coca-Cola's own currency effects from markets outside the U.S. are already inside its dollar revenue, so part of the operating variance is also currency.
- Q4 is derived as full year minus Q1–Q3, so restatements and acquisitions can distort it.
- SEC frames are aligned to calendar quarters; companies with non-December year ends need extra care.
- A single budget rate per year is a simplification; real plans may use hedged or monthly rates.
- Next step: add business drivers (such as unit case volume, package volume and airline capacity) instead of relying on past revenue alone.

## How to run
Open `forecast_vs_actual.ipynb` in Google Colab or Jupyter, set `USER_AGENT` to your own name and email (the SEC requires it), then run all cells.
