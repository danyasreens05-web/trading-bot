# trading-bot
# Binance Futures Testnet Trading Bot

A command-line tool for placing **MARKET** and **LIMIT** orders on the
[Binance USDT-M Futures Testnet](https://testnet.binancefuture.com)
using the [`python-binance`](https://github.com/sammchardy/python-binance)
library.

## Project Overview

This project provides a small, well-structured CLI application that:

1. Accepts order parameters from the command line.
2. Validates them thoroughly before touching the network.
3. Places the order on Binance USDT-M Futures Testnet.
4. Prints a clean summary before and after execution.
5. Logs every request, response, and failure to `logs/trading.log`.

It is built to be read and extended easily — logic is separated into
small modules with a single responsibility each, rather than one large
script.

## Features

- Supports **MARKET** and **LIMIT** orders, **BUY** and **SELL** sides.
- Argument parsing via `argparse`, including `--help` and usage examples.
- Thorough input validation with clear, actionable error messages.
- Colored terminal output (green for success, red for errors) via `colorama`.
- Structured logging to `logs/trading.log` with timestamps — API secrets
  are never logged.
- Graceful handling of validation errors, authentication failures,
  network timeouts, connection errors, and unexpected exceptions.
  **The CLI never crashes with a raw Python traceback.**
- No hardcoded credentials — API keys are loaded from a local `.env` file.

## Folder Structure

```
trading_bot/
├── bot/
│   ├── __init__.py          # Package marker / exports
│   ├── client.py            # Builds the authenticated Binance client
│   ├── orders.py             # Order placement business logic
│   ├── validators.py         # Input validation logic
│   └── logging_config.py     # Centralized logging setup
├── logs/                     # Created automatically; holds trading.log
├── config.py                 # Loads settings/credentials from .env
├── cli.py                    # CLI entry point (argument parsing + output)
├── requirements.txt          # Python dependencies
├── .env.example               # Template for required environment variables
└── README.md
```

## Installation

Requires **Python 3.11+**.

```bash
cd trading_bot
python3 -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Environment Variables

Copy the example file and fill in your Binance **Futures Testnet** API
credentials:

```bash
cp .env.example .env
```

`.env` contents:

| Variable              | Required | Description                                      |
|-----------------------|----------|---------------------------------------------------|
| `BINANCE_API_KEY`      | Yes      | Your Binance Futures Testnet API key               |
| `BINANCE_API_SECRET`   | Yes      | Your Binance Futures Testnet API secret            |
| `BINANCE_TESTNET_URL`  | No       | Overrides the default testnet base URL             |

## Setup

1. Create a Binance Futures Testnet account and generate API keys at
   https://testnet.binancefuture.com.
2. Fund your testnet futures wallet with test USDT (available from the
   testnet faucet on the same site).
3. Populate `.env` with your key and secret as described above.
4. Verify the CLI is wired up correctly:

   ```bash
   python cli.py --help
   ```

## Running Examples

### Market Order Example

Buy `0.01` BTC at the current market price:

```bash
python cli.py --symbol BTCUSDT --side BUY --type MARKET --quantity 0.01
```

### Limit Order Example

Sell `0.5` ETH at a limit price of `3200.50` USDT:

```bash
python cli.py --symbol ETHUSDT --side SELL --type LIMIT --quantity 0.5 --price 3200.50
```

### Sample Output

```
-----------------------------
ORDER SUMMARY
-----------------------------
Symbol:     BTCUSDT
Side:       BUY
Order Type: MARKET
Quantity:   0.01
Price:      N/A (MARKET)

-----------------------------
ORDER RESPONSE
-----------------------------
Order ID:          123456789
Status:            FILLED
Executed Quantity: 0.01
Average Price:     67321.10

SUCCESS
```

## Logging

- All logs are written to `logs/trading.log`, created automatically on
  first run.
- Each entry includes a timestamp, log level, source module, and message.
- Logged events include: outgoing request parameters, raw API responses,
  validation failures, network failures, and unexpected exceptions.
- **API secrets are never logged** — only non-sensitive order parameters
  and responses are written to the log file.
- Warnings and errors are also echoed to the console; informational logs
  are file-only to keep terminal output focused on the order summary.

## Error Handling

The CLI is designed to never crash with a raw traceback. Instead, it
catches and reports:

- **Invalid CLI arguments** — caught by `argparse` itself (e.g. missing
  required flags, invalid `--side`/`--type` choices).
- **Validation failures** — empty symbol, invalid side/type, non-positive
  quantity or price, missing price on LIMIT orders.
- **Authentication failures** — missing or invalid API credentials.
- **Binance API errors** — rejected orders, insufficient balance, invalid
  symbol, etc., surfaced with the exchange's error code and message.
- **Network issues** — request timeouts and connection errors are caught
  and reported with a friendly message.
- **Unexpected exceptions** — a final catch-all logs the full traceback
  to `logs/trading.log` (for debugging) while printing a short,
  user-friendly message to the console.

In every failure case, the program prints `FAILED` and exits with a
non-zero status code; on success it prints `SUCCESS` and exits `0`.

## Assumptions

- The user has already created a Binance Futures Testnet account and
  generated API keys specifically for the testnet (these are separate
  from live Binance account keys).
- Orders use `GTC` (Good 'Til Canceled) as the time-in-force policy for
  LIMIT orders, since the assignment did not specify otherwise.
- Quantity precision/lot-size rules (e.g. minimum quantity increments
  per symbol) are enforced by the Binance API itself; the CLI validates
  only that quantity and price are positive numbers, not exchange-specific
  precision rules.
- The tool is single-order-per-invocation by design — it is a CLI
  utility, not a long-running bot process.
