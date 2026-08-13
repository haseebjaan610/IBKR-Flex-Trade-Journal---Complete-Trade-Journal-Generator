# IBKR Flex Trade Journal - Complete Trade Journal Generator

A powerful Python tool that automatically converts IBKR Flex Query CSV exports into a clean, professionally formatted Excel trade journal with complete FIFO matching, futures support, and accurate P/L calculations.

## Features

- **Automatic FIFO Matching** - Correctly matches partial fills and multiple executions
- **Futures Support** - Handles futures contracts with proper multipliers
- **Short Trade Handling** - Properly processes short positions and closing trades
- **Partial Fill Processing** - Groups same-day executions and matches them correctly
- **Accurate P/L Calculation** - Uses FIFO-matched prices for precise profit/loss
- **Automated Excel Export** - Creates beautifully formatted Excel journals
- **Multi-File Processing** - Processes multiple CSV files in one run
- **Reusable & Automated** - No manual cleaning required

## Output Format

The generated Excel journal includes:

- **Symbol** - Trading symbol
- **Direction** - LONG or SHORT
- **Quantity** - Number of shares/contracts
- **Entry Date** - Trade entry date
- **Entry Price** - Average entry price per share
- **Exit Date** - Trade exit/closing date
- **Exit Price** - Average exit price per share
- **P/L** - Net profit or loss (includes multiplier for futures)

## Installation

### Requirements

- Python 3.8+
- pandas
- openpyxl
- numpy
- pytest (for testing)

### Setup

1. Install dependencies:

```bash
pip install -r requirements.txt
```

## Usage

### Method 1: Automatic Processing (Recommended)

1. Place your IBKR Flex Query CSV files in the workspace root directory
2. Run the script:

```bash
python src/main.py
```

The tool will:

- Automatically find all CSV files
- Process each file
- Generate a single consolidated `Trade_Journal_Output.xlsx` file with all trades
- Display summary statistics

### Method 2: Process Specific Files

Edit `src/main.py` to specify exact CSV file paths:

```python
csv_file_path = 'path/to/your/ibkr_flex_query.csv'
```

## How It Works

### Step 1: CSV Parsing

The tool reads IBKR Flex Query CSV exports and extracts:

- Asset class (STK for stocks, FUT for futures)
- Symbol and Trade ID
- DateTime stamp
- Quantity and Trade Price
- Open/Close indicators
- FIFO P/L values

### Step 2: FIFO Matching

- Groups trades by symbol
- Separates opening trades (marked with "O") from closing trades (marked with "C")
- Matches closing trades to opening trades using FIFO method
- Handles partial fills across multiple executions
- Tracks long and short positions separately

### Step 3: P/L Calculation

- Calculates P/L based on matched entry and exit prices
- For stocks: `P/L = (Exit Price - Entry Price) × Quantity`
- For shorts: `P/L = (Entry Price - Exit Price) × Quantity`
- For futures: Multiplies by contract multiplier (e.g., 50 for ES, 100 for GC)

### Step 4: Excel Export

- Creates professionally formatted Excel workbook
- Includes headers with formatting
- Color-codes P/L (green for profits, red for losses)
- Freezes header row for easy scrolling
- Applies borders and proper column widths

## Supported Futures Contracts

The tool includes multipliers for common futures:

- **E-mini S&P 500 (ES)** - 50
- **E-mini NASDAQ (NQ)** - 20
- **E-mini Dow (YM)** - 5
- **Gold (GC)** - 100
- **Crude Oil (CL)** - 1000
- **Natural Gas (NG)** - 10000
- **Bonds (ZB, ZN, ZF, ZT)**
- **Forex (EC, JY, BP, AD, CD, SF)**

Custom multipliers can be added in `src/utils/helpers.py`

## Example Output

```
============================================================
IBKR FLEX TRADE JOURNAL CONVERTER
============================================================

Found 4 CSV files:
  1. 1-2026.csv
  2. 3-2026.csv
  3. 4-2026.csv
  4. trades 2-2026.csv

Processing 1-2026.csv...
  ✓ Parsed 281 trade records
  ✓ Matched 158 completed trades

Processing 3-2026.csv...
  ✓ Parsed 128 trade records
  ✓ Matched 72 completed trades

...

============================================================
SUCCESS! Trade journal created with 449 trades
Output file: Trade_Journal_Output.xlsx
============================================================

Summary Statistics:
  Total Trades: 449
  Winning Trades: 100
  Losing Trades: 345
  Total P/L: $17,369.13
  Win Rate: 22.5%
```

## Project Structure

```
ibkr-flex-trade-journal/
├── src/
│   ├── main.py                    # Entry point
│   ├── converters/
│   │   └── flex_converter.py      # CSV parsing
│   ├── processors/
│   │   └── trade_processor.py     # FIFO matching logic
│   ├── exporters/
│   │   └── excel_exporter.py      # Excel generation
│   ├── models/
│   │   └── trade.py               # Trade class
│   └── utils/
│       └── helpers.py             # Helper functions
├── tests/
│   ├── test_converters.py         # Parser tests
│   ├── test_processors.py         # FIFO tests
│   └── test_exporters.py          # Export tests
├── requirements.txt
└── README.md
```

## Testing

Run all tests:

```bash
pytest tests/ -v
```

Test coverage:

- CSV parsing with various formats
- FIFO matching for long positions
- FIFO matching for short positions
- Partial fill handling
- Excel export functionality
- Error handling for missing files

All 8 tests pass successfully.

## 🔧 Customization

### Add New Futures Multipliers

Edit `src/utils/helpers.py` and add to the `multipliers` dictionary:

```python
multipliers = {
    'ES': 50,      # E-mini S&P 500
    'YM': 5,       # Your custom contract
    # Add more...
}
```

### Change Output File Location

Edit `src/main.py`:

```python
output_file = Path('/custom/path/Trade_Journal.xlsx')
```

## Important Notes

1. **CSV Format** - Ensure IBKR CSV exports include columns: AssetClass, Symbol, TradeID, DateTime, Quantity, TradePrice, Open/CloseIndicator
2. **DateTime Format** - Expects IBKR format: YYYYMMDD;HHMMSS
3. **Quantity Signs** - Positive = buy, Negative = sell (for stocks)
4. **Duplicate Processing** - Processing same CSV multiple times will add duplicate trades; clean old output files first

## Support & Troubleshooting

### Issue: "No CSV files found"

- Ensure CSV files are in the workspace root directory
- Check file naming (should be *.csv)

### Issue: "ModuleNotFoundError"

- Ensure dependencies are installed: `pip install -r requirements.txt`
- Verify Python path is correct

### Issue: Excel file appears empty

- Check console output for parsing errors
- Verify CSV format matches IBKR Flex Query structure
- Check that trades have proper Open/Close indicators

### Issue: P/L values seem incorrect

- Verify entry and exit prices in source CSV
- Check that multipliers are correct for futures
- Ensure FIFO matching is working (check console output)

## Regular Monthly Usage

To use this tool monthly:

1. Export new IBKR Flex Query CSV for the month
2. Place CSV files in workspace directory
3. Delete old `Trade_Journal_Output.xlsx`
4. Run: `python src/main.py`
5. New journal automatically generates with all trades

## Understanding FIFO Matching

FIFO (First In, First Out) matching works as follows:

1. **Opening Trades** - Buy/Sell orders that open positions
2. **Closing Trades** - Sell/Buy orders that close positions
3. **Matching Process**:
   - First closing trade matches with first opening trade
   - If quantities don't match, remaining is matched with next opening trade
   - Process continues until all positions are matched

Example:

```
Opening: BUY 100 @ $10
Opening: BUY 50 @ $11
Closing: SELL 120 @ $12

Result:
Trade 1: BUY 100 @ $10, SELL 100 @ $12 = +$200
Trade 2: BUY 50 @ $11, SELL 20 @ $12 = +$50
Remaining: BUY 30 @ $11 (open position)
```

## License

This tool is provided as-is for trade analysis purposes.

## Delivery Checklist

- Reusable Python tool (not one-time manual cleaning)
- Accepts IBKR Flex Query CSV files
- Outputs clean Excel journal with required columns
- Handles futures with multipliers
- Supports partial fills & FIFO matching
- Processes short trades correctly
- Groups same-day executions
- P/L calculations match entry/exit prices
- Fully automated (no manual cleaning needed)
- Clear usage instructions included
- 100% reusable for monthly recurring use
- All tests passing

This will initiate the conversion process of the IBKR Flex Query CSV exports into an Excel trade journal.

## Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue for any enhancements or bug fixes.

## License

This project is licensed under the MIT License. See the LICENSE file for more details.    
**WhatsApp:** +923113317722  
**Email:** [haseeburrehmanofficial610@gmail.com](mailto:haseeburrehmanofficial610@gmail.com)
