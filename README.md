# Cryptocurrency Tracker

A Flask-based web app to track cryptocurrency holdings, current value, and profit/loss using live price data from the CoinGecko API.

---

## Table of Contents

- [1) Project Overview](#1-project-overview)
- [2) Features](#2-features)
- [3) Tech Stack](#3-tech-stack)
- [4) Current Project Structure](#4-current-project-structure)
- [5) How It Works](#5-how-it-works)
- [6) API Endpoints Used](#6-api-endpoints-used)
- [7) Installation (Windows)](#7-installation-windows)
- [8) Run the App](#8-run-the-app)
- [9) Application Behavior](#9-application-behavior)
- [10) Data Model](#10-data-model)
- [11) Calculations](#11-calculations)
- [12) Error Handling](#12-error-handling)
- [13) Limitations](#13-limitations)
- [14) Security Notes](#14-security-notes)
- [15) Troubleshooting](#15-troubleshooting)
- [16) Suggested Improvements](#16-suggested-improvements)
- [17) License](#17-license)
- [18) Disclaimer](#18-disclaimer)

---

## 1) Project Overview

This application allows a user to:

- Add coins to a portfolio
- Edit existing entries
- Delete entries
- Fetch current USD prices from CoinGecko
- View:
  - Total investment
  - Total current value
  - Net profit/loss
  - Best and worst performing coin by absolute P/L

Portfolio data is stored locally in `portfolio.json`.

---

## 2) Features

- **Web UI with Flask** (`/` route)
- **Persistent local storage** with JSON file (`portfolio.json`)
- **Coin ID validation** using CoinGecko `/coins/list`
- **Bulk price fetching** using CoinGecko `/simple/price`
- **Live P/L analytics** for each coin
- **Portfolio summary metrics**:
  - Total investment
  - Total value
  - Net P/L
  - Best/Worst coin

---

## 3) Tech Stack

- **Python 3.9+** (recommended 3.10+)
- **Flask**
- **Requests**
- **JSON file storage** (no database)

---

## 4) Current Project Structure

Expected minimal structure:

```text
Crytocurrency Tracker/
├── main.py
├── portfolio.json          # auto-created/updated
└── templates/
    └── index.html
└── static/
    └── style.css
```

> Note: Folder name currently appears as `Crytocurrency Tracker` (spelling typo).  
> You can keep it as-is or rename to `Cryptocurrency Tracker`.

---

## 5) How It Works

### Main flow (`main.py`)

1. Load portfolio from `portfolio.json`
2. Handle form action (`add`, `edit`, `delete`)
3. Validate and persist updated data
4. Fetch latest prices for all portfolio coins
5. Compute investment/value/P&L stats
6. Render `index.html` with full data

### Key functions

- `load_portfolio()`  
  Reads JSON and returns list; returns `[]` on file not found/invalid JSON.

- `save_portfolio(portfolio)`  
  Writes portfolio list to `portfolio.json`.

- `get_coin_list()`  
  Fetches full CoinGecko coin IDs and caches in memory (`coin_cache`).

- `is_valid_coin(coin)`  
  Checks if provided coin ID exists in cached list.

- `fetch_prices(coins)`  
  Fetches current USD prices for all coins in one request.

---

## 6) API Endpoints Used

### 1) Coin list (validation)
`GET https://api.coingecko.com/api/v3/coins/list`

Used to validate user input coin IDs (example valid IDs: `bitcoin`, `ethereum`, `solana`).

### 2) Current prices (bulk)
`GET https://api.coingecko.com/api/v3/simple/price?ids=<comma-separated-ids>&vs_currencies=usd`

Returns current USD price by coin ID.

---

## 7) Installation (Windows)

### 1) Open terminal in project folder

```powershell
cd "d:\PYTHON\Crytocurrency Tracker"
```

### 2) Create virtual environment

```powershell
py -m venv .venv
```

### 3) Activate virtual environment

```powershell
.\.venv\Scripts\Activate.ps1
```

### 4) Install dependencies

```powershell
pip install flask requests
```

(Optional) freeze dependencies:

```powershell
pip freeze > requirements.txt
```

---

## 8) Run the App

```powershell
python main.py
```

Then open:

- `http://127.0.0.1:5000/`

---

## 9) Application Behavior

### Add coin

Form must include:
- `coin` (CoinGecko coin ID)
- `quantity` (float)
- `buy_price` (float)
- submit field containing `add`

Behavior:
- Validates numeric inputs
- Validates coin ID against CoinGecko list
- If coin already exists:
  - adds quantity
  - replaces `buy_price` with new value
- Saves and redirects to `/`

### Edit coin

Form must include:
- `coin`
- `quantity`
- `buy_price`
- submit field containing `edit`

Behavior:
- Updates matching coin entry
- Saves and redirects

### Delete coin

Form must include:
- `coin`
- submit field containing `delete`

Behavior:
- Removes matching coin from list
- Saves and redirects

---

## 10) Data Model

Each portfolio item:

```json
{
  "coin": "bitcoin",
  "quantity": 0.5,
  "buy_price": 40000.0
}
```

Computed (not persisted by `save_portfolio` unless manually included before saving):
- `current_price`
- `value`
- `profit_loss`

---

## 11) Calculations

For each coin:
- `investment = quantity * buy_price`
- `current_value = quantity * current_price`
- `profit_loss = current_value - investment`

Portfolio totals:
- `total_investment = sum(investment)`
- `total_value = sum(current_value)`
- `net_profit_loss = total_value - total_investment`

Best/Worst coin:
- Highest/lowest absolute `profit_loss`

Fallback:
- If live price unavailable, app uses `buy_price` as `current_price`.

---

## 12) Error Handling

Current handling in code:

- `load_portfolio()` handles:
  - missing file
  - invalid JSON

- API calls (`get_coin_list`, `fetch_prices`) wrapped in broad `try/except`:
  - on failure, returns empty list/dict

- Add form handles invalid numeric input and shows:
  - `❌ Invalid number input`

- Invalid coin ID shows:
  - `❌ '<coin>' is not a valid cryptocurrency`

---

## 13) Limitations

- Uses broad `except:` (can hide real errors)
- No database; JSON file may not scale
- No authentication/user accounts
- No transaction history (only latest position values)
- `buy_price` overwrite behavior on add for existing coin (no weighted average)
- Coin list cache is in-memory only (resets on app restart)
- `debug=True` in production is unsafe

---

## 14) Security Notes

- Do not run Flask debug mode in production
- Validate and sanitize all form inputs in template/backend
- Consider CSRF protection (Flask-WTF) for forms
- Use proper WSGI server for deployment (e.g., Gunicorn/Waitress)
- Add request rate limiting if deployed publicly

---

## 15) Troubleshooting

### App does not start
- Ensure Flask is installed:
  ```powershell
  pip install flask
  ```

### Template error
- Confirm `templates/index.html` exists and is valid HTML/Jinja.

### Coin always invalid
- Ensure internet connectivity.
- Coin must be CoinGecko **ID**, not ticker symbol in many cases.
  - Example: use `bitcoin`, not `BTC`.

### Prices not updating
- CoinGecko may rate-limit or temporary fail.
- Retry after a short time.

### JSON errors
- Delete or fix malformed `portfolio.json`.
- App auto-recovers to empty portfolio on JSON decode failure.

---

## 16) Suggested Improvements

- Replace broad `except:` with specific exceptions
- Add `requirements.txt`
- Add `.env` config and app settings
- Add weighted average buy price logic
- Add transaction history (buy/sell) instead of overwrite model
- Add unit tests with `pytest`
- Add logging module
- Add pagination/search for large portfolios
- Add support for multiple fiat currencies
- Add charting (historical price trend)

---

## 17) License

Add your preferred license, for example:

`MIT License`

---

## 18) Disclaimer

This project is for educational and informational use only.  
It does not provide financial advice. Use market data and calculations at your own risk.