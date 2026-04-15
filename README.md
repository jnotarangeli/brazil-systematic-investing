# brazil-systematic-investing

Construction and empirical evaluation of a Fama-French style value factor (HML) for the Brazilian equity market. The pipeline builds a monthly panel of Brazilian equities from WRDS Compustat Global and Datastream, applies the standard double-sort methodology with emerging-market adjustments, and evaluates the factor against Brazilian market benchmarks.

## Research Hypothesis

The value premium documented in the US (Fama & French 1993) should persist in emerging markets, but standard implementation requires adjustments for data sparsity, crisis-period B/M instability, and the thinner cross-section of Brazilian equities. Size-conditional B/M breakpoints and mild winsorization produce a more stable factor that captures persistent cross-sectional value effects rather than distressed small-cap exposure.

## Data Sources

All raw data files are stored locally and excluded from version control.

| Source | WRDS Table / Dataset | Description |
|---|---|---|
| Compustat Global | `comp.g_funda` (annual) | Book equity and fundamentals for Brazilian firms (2000–2025) |
| WRDS Datastream | Datastream monthly via WRDS | Monthly returns, prices, market cap |
| Compustat Global Security | `comp.g_security` | ISIN bridge table linking Compustat gvkeys to Datastream securities |
| Brazilian Benchmarks | BRBOVES (Ibovespa), BRIBXIN (IBrX-100) | Market return benchmarks for factor regressions |

## Pipeline Overview

### `brazil_fundamentals.ipynb` — Data Construction (run first)
1. Pull annual fundamentals from Compustat Global (2000–2025)
2. Pull monthly returns, prices, and market cap from WRDS Datastream — restricted to firms with a Compustat record via `comp.g_security` ISIN bridge
3. Merge via ISIN with point-in-time alignment (6-month accounting lag)
4. Compute 24 fundamental characteristics and 9 market-scaled characteristics
5. Drop variables with >50% missing (`rd_me`, `rd_sale`, `emp_gr1`)
6. Save cleaned panel to `data/brazil_fundamental_panel.parquet`

### `value.ipynb` — HML Factor Construction & Evaluation
1. **Data Loading** — Load cleaned fundamentals panel
2. **June Rebalancing** — Each June, snapshot the most recent B/M and market equity per firm
3. **Double Sort:**
   - Split firms into Small / Big on the size median
   - Within each size group, compute B/M 30th/70th percentiles (size-conditional breakpoints)
   - Form six value-weighted portfolios: SL, SM, SH, BL, BM, BH
4. **HML Construction:** `HML = ½(SH + BH) − ½(SL + BL)`; held July(t) → June(t+1)
5. **Emerging Market Adjustments:** B/M winsorization; minimum portfolio size constraint; size-conditional breakpoints to prevent the factor from collapsing into distressed small-cap exposure during crises
6. **Performance Evaluation:** Annualized return, volatility, Sharpe ratio; cumulative return chart
7. **Factor Regressions:** `r_HML,t = α + β·r_Market,t + ε_t` vs. Ibovespa and IBrX-100

## Key Results

| Strategy | Ann. Return | Ann. Vol | Sharpe | Terminal $1 |
|---|---|---|---|---|
| Ibovespa | 14.1% | 22.8% | 0.62 | $14.5 |
| HML (Long–Short) | 8.4% | 13.4% | 0.63 | $5.8 |
| Ibovespa + HML | 22.5% | 25.6% | 0.88 | $88.8 |

The HML factor achieves a Sharpe ratio comparable to the market with materially lower volatility. Its low correlation with the Ibovespa contributes meaningful diversification: combining the two raises the portfolio Sharpe from 0.62 to 0.88.

## Requirements

- **WRDS account** with access to Compustat Global and Datastream
- Python 3.9+
- Key packages: `pandas`, `numpy`, `matplotlib`, `wrds`, `sqlalchemy`

## How to Run

1. Ensure WRDS credentials are configured (`~/.pgpass` or environment variables)
2. Run `brazil_fundamentals.ipynb` to build the firm-level panel (run once; output cached locally)
3. Run `value.ipynb` end-to-end for factor construction, performance evaluation, and regressions

## References

- Fama, E. F., & French, K. R. (1993). Common Risk Factors in the Returns on Stocks and Bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Jensen, T. I., Kelly, B. T., & Pedersen, L. H. (2023). Is There a Replication Crisis in Finance? *Journal of Finance*, 78(5), 2465–2518.
