# CLAUDE.md — brazil-systematic-investing

## What This Project Is
A Fama-French HML value factor construction pipeline for the Brazilian equity market. Pulls annual fundamentals from Compustat Global and monthly return data from WRDS Datastream, merges via ISIN with point-in-time alignment, applies size-conditional double sorts with emerging-market adjustments, and evaluates the resulting long-short factor against Brazilian market benchmarks (Ibovespa, IBrX-100).

## Python Stack
- `pandas`, `numpy` — data manipulation
- `wrds`, `sqlalchemy` — WRDS database access (Compustat Global, Datastream)
- `matplotlib` — charting
- `pathlib` — file I/O for parquet caching

## Data Sources
Raw data lives **locally only** and is never committed to version control:
- `data/brazil_fundamental_panel.parquet` — output of `brazil_fundamentals.ipynb`
- `brazil_characteristics.csv` — cleaned fundamentals used by `value.ipynb`
- `brazil_hml.csv` — constructed HML factor time series
- Compustat Global and Datastream pulled live from WRDS

## Files to NEVER Commit
*.parquet, *.csv, *.xlsx, *.png, *.jpg, *.pdf, *.svg, .env
Data/, data/, figures/, output/, Old/, .venv/

## Coding Conventions
- Vectorized operations only — no `iterrows()` or Python loops over DataFrames
- Portfolio formation always in June; holding period July(t) → June(t+1)
- B/M breakpoints computed size-conditionally (within Small and Big groups separately)
- Use `merge_asof` for point-in-time accounting data alignment
- Value-weighted portfolios using beginning-of-month market equity

## Session Start
1. Run `brazil_fundamentals.ipynb` first — builds the firm-level panel (WRDS required)
2. Run `value.ipynb` — loads the cleaned panel and constructs the HML factor
3. Both notebooks cache outputs locally; subsequent runs skip WRDS queries
4. WRDS connection requires `~/.pgpass` or inline credentials
