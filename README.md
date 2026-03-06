# Stock Analysis for the Course of UCB MFE Python Pre-Program 2026

**Coder**: Reza Zamani  

## Overview
Here we want to have comprehensive data cleaning, quality assessment, and automated risk analysis using portfolio of stocks. This project has three main components:

1. **Data Cleaning** (`01_data_cleaning.ipynb`)
2. **Single Stock Risk Analysis Template** (`02_stock_report_template.ipynb`)
3. **Automated Multi-Stock Analysis** (`03_run_all_stocks.py`)

## Project Structure: 

```
Homeworks/HW2/Reza_Zamani/
├── 01_data_cleaning.ipynb          # Data cleaning and validation notebook
├── 02_stock_report_template.ipynb  # Parameterized stock report template
├── 03_run_all_stocks.py            # Automated report generation script
├── data3.db                          # SQLite database (input/output)
├── requirements.txt                 # Python dependencies
├── reports/                         # Generated reports (created by script)
│   └── YYYY-MM-DD/
│       ├── {TICKER}_report.ipynb
│       ├── portfolio_summary.csv
│       ├── risk_return_profile.png
│       ├── total_returns_by_stock.png
│       └── failed_stocks.txt (if any)
└── README.md                        # This file
```


## Part 1: Data Cleaning & Validation

**Notebook**: `01_data_cleaning.ipynb`

### Objectives
- Load raw OHLC data from `data3.db`
- Detect and report data quality issues
- Clean the data systematically
- Validate cleaned data
- Save to `ohlc_hw_cleaned` table

### Key Features
- **Missing Data Detection**: Identifies missing values and trading day gaps using pandas business day logic
- **Outlier Detection**: Uses 20-day rolling statistics (adapts to market volatility)
- **Inconsistency Detection**: Validates OHLC relationships (low ≤ open ≤ high, low ≤ close ≤ high)
- **Data Quality Report**: Comprehensive CSV report with issue severity ratings
- **adj_close Handling**: Properly handles adjusted close prices with forward-fill and backfill

### Cleaning Strategy
1. Clean ticker names (remove HTML tags)
2. Handle missing values (forward-fill prices, fill volume with 0)
3. Remove duplicates (by ticker and date)
4. Handle outliers (fix impossible price relationships)
5. Fix OHLC inconsistencies


### Output
- `ohlc_hw_cleaned` table in `data3.db`
- `data_quality_report.csv` (if issues detected)

## Part 2: Single Stock Risk Analysis

**Notebook**: `02_stock_report_template.ipynb`

### Objectives
Create a parameterized template that generates comprehensive risk reports for individual stocks one by one.

### Parameters (Papermill-compatible)
```python
ticker = "AAPL"              # is here an example: Stock ticker to analyze
start_date = "2024-01-01"    # Analysis start date
end_date = "2024-12-31"      # Analysis end date
```

**Note**: The parameters cell is tagged as "parameters" for papermill execution.

### Risk Metrics Calculated
- **Daily Returns**: Log returns for statistical analysis
- **Rolling Volatility**: 20-day annualized volatility
- **Maximum Drawdown**: Cumulative drawdown from peak
- **Sharpe Ratio**: Risk-adjusted return (assuming 2% risk-free rate)

### Summary Table Covers:
- Total Return
- Annualized Return
- Annualized Volatility
- Sharpe Ratio
- Maximum Drawdown
- Best/Worst Day
- Positive Days %
- Average Up/Down Day

### Usage

**Interactive**:
```bash
jupyter notebook 02_stock_report_template.ipynb
# Modify parameters and run all cells
```

**Automated (with papermill)**:
```bash
papermill 02_stock_report_template.ipynb 
 -   various stocks 
    -p start_date 2024-01-01 \
    -p end_date 2024-12-31
```

## Part 3: Automated Multi-Stock Analysis

**Script**: `03_run_all_stocks.py`

### Objectives
Automatically generate risk reports for all stocks in the cleaned database.

### Features
- **Auto Date Range**: Determines last 6 complete months from database
- **Progress Tracking**: tqdm progress bar with status updates
- **Error Handling**: Graceful error handling with detailed logging
- **Portfolio Summary**: Consolidated metrics across all stocks
- **Risk-Return Visualization**: Scatter plot colored by Sharpe ratio

### Workflow
1. Load all tickers from `ohlc_hw_cleaned`
2. Determine date range (last 6 complete months)
3. Create output directory: `reports/YYYY-MM-DD/`
4. For each ticker:
   - Execute `02_stock_report_template.ipynb` with papermill
   - Extract and store risk metrics
   - Log any errors
5. Generate portfolio summary CSV
6. Create risk-return scatter plot
7. Display top/bottom 10 stocks by Sharpe ratio

### Usage
```bash
python 03_run_all_stocks.py
```

### Output
- `reports/YYYY-MM-DD/{TICKER}_report.ipynb` - Individual reports
- `reports/YYYY-MM-DD/portfolio_summary.csv` - Consolidated metrics
- `reports/YYYY-MM-DD/risk_return_profile.png` - Visualization
- `reports/YYYY-MM-DD/total_returns_by_stock.png` - Visualization
- `reports/YYYY-MM-DD/avg_volatility_by_stock.png` - Visualization
- `reports/YYYY-MM-DD/failed_stocks.txt` - Error log (if failures)


## Key Design Decisions

### 1. Log Returns vs Simple Returns
- **Choice**: Log returns
- **Rationale**: Additive over time, better for statistical analysis

### 2. Rolling vs Global Statistics for Outliers
- **Choice**: Rolling 20-day window
- **Rationale**: Adapts to changing market conditions and volatility regimes

### 3. adj_close Handling
- **Choice**: Forward-fill, then backfill for leading NaNs
- **Rationale**: Maintains time series continuity while preserving split/dividend adjustments

### 4. Date Range for Multi-Stock Analysis
- **Choice**: Last 6 complete months
- **Rationale**: Ensures all stocks have comparable time periods

### 5. Error Handling in Automation
- **Choice**: Continue on error, log failures
- **Rationale**: Maximize successful reports, provide detailed error tracking

## Performance Considerations

- **Data Loading**: Uses SQLite queries with date filtering for efficiency
- **Parallel Processing**: Sequential execution (could be parallelized with multiprocessing)
- **Memory Management**: Processes one stock at a time, clears matplotlib figures
- **Report Generation**: Papermill executes notebooks in isolated kernels

## Dependencies

Core libraries:
- `pandas>=1.3.4` - Data manipulation
- `numpy>=1.26.0` - Numerical computations
- `matplotlib>=3.5.0` - Visualizations
- `papermill>=2.3.3` - Notebook automation
- `tqdm>=4.66.0` - Progress bars
- `python-dateutil>=2.8.0` - Date calculations
- `seaborn>=0.11.0` - Enhanced visualizations

## Testing

To verify the complete workflow:

```bash
# 1. Run data cleaning
jupyter nbconvert --to notebook --execute 01_data_cleaning.ipynb

# 2. Test single stock report
papermill 02_stock_report_template.ipynb test_report.ipynb \
    -p ticker AAPL -p start_date 2024-01-01 -p end_date 2024-12-31

# 3. Run automated analysis
python 03_run_all_stocks.py
```

## Expected Results

- **Data Cleaning**: ~30 unique tickers from ~400K rows
- **Quality Issues**: Detection of missing values, outliers, and inconsistencies
- **Individual Reports**: Beautiful visualizations with comprehensive metrics
- **Portfolio Analysis**: Risk-return profile showing stock positioning
- **Execution Time**: ~2-5 minutes for 30 stocks (depends on system)

## Troubleshooting

### Common Issues

1. **"No data found for ticker X"**
   - Check date range overlaps with available data
   - Verify ticker exists in `ohlc_hw_cleaned`

2. **Papermill kernel errors**
   - Ensure ipykernel is installed: `pip install ipykernel`
   - Verify kernel name matches: `python3`

3. **Memory errors with large datasets**
   - Process fewer stocks at a time
   - Increase system RAM or use cloud compute

4. **Visualization rendering issues**
   - Use `%matplotlib inline` in notebooks
   - Install proper backend: `pip install PyQt5`

## Future Enhancements

Potential improvements:
- Parallel processing for faster execution
- Interactive dashboard with Plotly/Dash
- Comparison with benchmark indices (S&P 500)
- Correlation analysis and portfolio optimization
- Risk factor decomposition (Fama-French)
- Real-time data updates via API integration

## References

- Lecture 3: Real World Data Challenge
- Lecture 4: Stock Analysis Template
- Lecture 4: Running Analysis at Scale

## License

This project is part of the UCB MFE Python Pre-Program coursework.

---

**Note**: This README assumes the cleaned database (`ohlc_hw_cleaned`) has been created by running Part 1. Always run notebooks in order: 01 → 02 → 03.

