# quantitative_investing

# Brazilian Value Factor Construction (HML)

## Overview

This repository implements the construction and empirical evaluation of
a **Value (High Minus Low, HML-style) factor** for the Brazilian equity
market using WRDS Datastream data.

The objective is to:

-   Construct size and book-to-market sorted portfolios
-   Form a long-short value portfolio
-   Evaluate its performance and robustness
-   Test market exposure via regression against Brazilian equity indices

This project follows the Fama-French (1993) methodology with
adjustments appropriate for emerging market data.

------------------------------------------------------------------------

## Data Sources

All data are obtained via **WRDS Datastream**.

### Equity Data

-   Market capitalization
-   Book equity
-   Monthly returns

### Market Benchmarks

-   **BRBOVES** (Bovespa / Ibovespa index)
-   **BRIBXIN** (IBrX-100 index)
-   BRIBX50 (robustness check)

MSCI Brazil was not available due to licensing restrictions.

------------------------------------------------------------------------

## Methodology

### 1. Timing Convention

Portfolios are formed annually in **June of year t** and held from:

July t → June t+1

This ensures: - No look-ahead bias - Accounting data are publicly
available - Consistency with Fama--French timing conventions

------------------------------------------------------------------------

### 2. Double Sorting Procedure

Each June:

1.  **Size Sort**
    -   Split firms into Small (S) and Big (B)
    -   Breakpoint: median market capitalization
2.  **Book-to-Market Sort**
    -   Within each size group, sort into:
        -   Low (L) --- bottom 30%
        -   Medium (M) --- middle 40%
        -   High (H) --- top 30%

This produces six portfolios:

          Low   Med   High
  ------- ----- ----- ------
  Small   SL    SM    SH
  Big     BL    BM    BH

------------------------------------------------------------------------

### 3. Factor Construction

HML = 1/2 (SH + BH) − 1/2 (SL + BL)

The factor is:

-   Long high book-to-market stocks
-   Short low book-to-market stocks
-   Neutral with respect to size

Portfolios are value-weighted.

------------------------------------------------------------------------

## Emerging Market Adjustments

During crisis periods (e.g., 2015–2016), book-to-market (B/M) distributions become heavily right-skewed.

To address this instability, the following adjustments were implemented:

- Mild winsorization applied to B/M ratios to reduce extreme right-tail influence  
- Minimum portfolio size constraint enforced to avoid thin long–short legs  
- Size-conditional breakpoints: B/M breakpoints are computed separately within Small and Big size groups rather than on the pooled cross-section  
- Sensitivity checks performed using alternative breakpoint definitions  

The size-conditional sorting ensures that value variation is measured *within* size categories, preventing the value factor from mechanically becoming a distressed small-cap factor during crisis episodes.

These steps improve portfolio balance, enhance stability of factor loadings, and reduce distortion of the long–short value signal.

------------------------------------------------------------------------

## Empirical Results

The constructed value factor delivers economically meaningful performance:

| Strategy              | Ann Return | Ann Vol | Sharpe | Terminal $1 |
|-----------------------|-----------|---------|--------|-------------|
| Ibovespa              | 14.1%     | 22.8%   | 0.62   | $14.5       |
| HML (Long–Short)      | 8.4%      | 13.4%   | 0.63   | $5.8        |
| Ibovespa + HML        | 22.5%     | 25.6%   | 0.88   | $88.8       |

Key observations:

- The standalone HML factor achieves a Sharpe ratio of 0.63, comparable to the market, with materially lower volatility.
- The factor exhibits low correlation with the Ibovespa, contributing diversification benefits.
- Combining the value factor with the market portfolio increases the Sharpe ratio to **0.88**, substantially improving risk-adjusted returns.
- Performance remains stable across subperiods, including crisis episodes.

The results suggest that the strategy captures persistent cross-sectional value effects rather than simple exposure to broad market risk.
Regression results:

r_HML,t = α + β r_Market,t + ε_t

-   Market beta estimated against BRBOVES and IBrX-100
-   Positive and economically meaningful alpha
-   Moderate market exposure during crisis periods

------------------------------------------------------------------------

## Files

- `value.ipynb` — Full construction pipeline
  - Data cleaning  
  - Breakpoint computation  
  - Portfolio formation  
  - Factor construction  
  - Performance evaluation  
  - Regression analysis  

- `brazil_characteristics.csv` — Processed firm-level dataset used in factor construction  
  - Contains size, book-to-market, and return variables  
  - Cleaned and aligned to June portfolio formation dates  
  - Serves as the intermediate dataset linking raw Datastream inputs to portfolio outputs
    
------------------------------------------------------------------------

## Reproducibility

To reproduce results:

1.  Ensure WRDS connection via `wrds` Python package
2.  Confirm access to Datastream library
3.  Update WRDS credentials
4.  Run notebook from top to bottom

------------------------------------------------------------------------

## Future Extensions

- Extend the framework to a multi-factor model by incorporating profitability and investment characteristics  
- Explicitly orthogonalize value relative to size and other risk factors  
- Implement rolling or adaptive breakpoints to account for structural market shifts  
- Evaluate interaction effects with macroeconomic variables and crisis regimes  
- Benchmark performance against international value indices and global factor models  
- Explore advanced portfolio construction techniques 
- Incorporate machine learning methods for feature selection and signal combination
