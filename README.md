# AutoFinder 🚗

> Simple, automated car inventory search with finance estimates and price tracking

AutoFinder is a GitHub Pages-hosted website that automatically searches nearby car inventory twice daily, normalizes results, calculates monthly payments, and tracks price changes — all without a database.

---

## Features

- ✅ **Automated Search**: Runs twice daily (07:30 & 19:30 CST) via GitHub Actions
- 💰 **Finance Estimates**: Calculate monthly payments with configurable parameters
- 📊 **Price Tracking**: Visual indicators (▲▼●) show price movements
- 🔍 **Smart Deduplication**: VIN-based matching prevents duplicates
- 📱 **Responsive UI**: Clean, accessible table interface
- 🏷️ **Sortable Columns**: Click headers to sort by price, mileage, etc.
- 🎯 **Budget Filtering**: Only shows affordable vehicles
- 🌐 **Static Hosting**: Runs on GitHub Pages — no servers needed

---

## Quick Start

### 1. Configure Search

Edit `config/app.config.json` with your ZIP code, budget, and preferences.

### 2. Add Secrets

Settings → Secrets → Actions:
- `GEMINI_API_KEY` (optional AI search)

### 3. Enable GitHub Pages

Settings → Pages → Source: **GitHub Actions**

### 4. Trigger Workflow

Actions tab → "Fetch Vehicle Inventory" → Run workflow

---

## Project Structure

```
buy-a-car/
├── .github/workflows/    # GitHub Actions
├── config/              # Configuration
├── data/                # Generated JSON data
├── scripts/             # Python backend
│   ├── fetch.py         # Main orchestrator
│   ├── models.py        # Data models
│   ├── finance.py       # Finance calculations
│   ├── normalize.py     # Deduplication
│   ├── price_tracker.py # Price changes
│   └── sources/         # Data sources
└── site/                # React frontend
    └── src/
        ├── App.tsx      # Main UI
        ├── types/       # TypeScript types
        └── api/         # Data fetching
```

---

## Local Development

### Quick Start (5 minutes)

See **[QUICKSTART.md](QUICKSTART.md)** for step-by-step instructions.

```bash
# Install dependencies
pip install -r requirements.txt
cd site && npm install && cd ..

# Set API keys
export GOOGLE_API_KEY='your-key'
export GOOGLE_CSE_ID='your-cse-id'
export GEMINI_API_KEY='your-key'

# Run with detailed logging
python run_local.py

# Or see demo without API keys
python demo_logging.py
```

### Features

- 📊 **Detailed HTTP Logging**: See every request with timing
- 🎨 **Color-Coded Output**: Status codes, durations, sizes
- ⏱️ **Progress Tracking**: Real-time updates for each stage
- 📈 **Timing Breakdown**: See where time is spent
- 🔍 **Debugging**: Verbose error messages

### Documentation

- **[QUICKSTART.md](QUICKSTART.md)** - Get started in 5 minutes
- **[LOCAL_SETUP.md](LOCAL_SETUP.md)** - Comprehensive guide with troubleshooting

---

## Tech Stack

- **Backend**: Python 3.11, Pydantic, httpx, Gemini API
- **Frontend**: React 18, TypeScript, Vite
- **Infrastructure**: GitHub Actions, GitHub Pages

---

## License

MIT
