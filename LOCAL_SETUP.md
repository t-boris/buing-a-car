# Local Development Setup

This guide explains how to run AutoFinder locally on your computer with detailed logging.

## Prerequisites

1. **Python 3.11+**
   ```bash
   python3 --version  # Should be 3.11 or higher
   ```

2. **Node.js 18+** (for frontend)
   ```bash
   node --version  # Should be 18 or higher
   ```

3. **API Keys**
   - Google Custom Search API key
   - Google Custom Search Engine ID
   - Gemini API key

## Installation

### 1. Clone and Install Dependencies

```bash
# Clone repository (if not already done)
git clone https://github.com/t-boris/buy-a-car.git
cd buy-a-car

# Install Python dependencies
pip install -r requirements.txt

# Install frontend dependencies
cd site
npm install
cd ..
```

### 2. Set Environment Variables

Create a `.env` file in the project root or export variables:

```bash
# Required API credentials
export GOOGLE_API_KEY='your-google-api-key-here'
export GOOGLE_CSE_ID='your-google-cse-id-here'
export GEMINI_API_KEY='your-gemini-api-key-here'
```

**Getting API Keys:**
- **Google Custom Search API**: https://developers.google.com/custom-search/v1/overview
- **Gemini API**: https://ai.google.dev/

### 3. Configure Search Parameters

Edit `config/app.config.json` to customize your search:

```json
{
  "zip": "60031",           // Your ZIP code
  "radius_miles": 15,       // Search radius
  "max_down_payment": 3000, // Budget constraints
  "max_monthly_payment": 450,
  "filters": {
    "min_year": 2018,
    "max_mileage": 90000,
    "include_makes": [       // Brands to search
      "Toyota", "Honda", "Hyundai"
    ]
  }
}
```

## Running Locally

### Option 1: Full Pipeline with Detailed Logs (Recommended)

Run the complete fetch pipeline with verbose logging:

```bash
python run_local.py
```

**What you'll see:**
- ✅ All HTTP requests with timing (Google Search, Gemini API, page fetches)
- 📊 Progress bars and status updates
- ⏱️ Duration for each stage
- 📈 Statistics and results summary
- 🎨 Color-coded output for easy reading

**Expected runtime:** 5-20 minutes depending on:
- Number of dealerships found
- Number of inventory pages
- Gemini API processing time

**Example output:**
```
================================================================================
                        AutoFinder Local Runner
================================================================================

ℹ Location: ZIP 60031, Radius: 15 miles
ℹ Makes: Toyota, Honda, Hyundai, Kia, Subaru, Mazda
ℹ Budget: $3000 down, $450/mo

[1/4] Finding Dealerships
    [1/32] Searching: car dealership Gurnee IL
      → GET  200    1.2s    15.2KB https://www.googleapis.com/customsearch/v1?q=...
        ✓ Found 10 results
    [2/32] Searching: used cars Gurnee IL
      → GET  200    0.8s    12.1KB https://www.googleapis.com/customsearch/v1?q=...
        ✓ Found 10 results
    ...
✓ Completed in 2m 15s

[2/4] Searching Inventory Pages
    [1/126] Searching inventory for: CarWise Gurnee
      → GET  200    1.1s    18.3KB https://www.googleapis.com/customsearch/v1?q=...
        ✓ Found 8 results
    ...
✓ Completed in 3m 42s

[3/4] Parsing with Gemini AI
    [1/23] Processing batch (8 pages)...
      → GET  200    0.5s   124.2KB https://www.carwisegurnee.com/used-vehicles/
      → GET  200    0.4s    98.1KB https://www.carwisegurnee.com/used-vehicles/page-2/
      → POST 200   12.3s    45.1KB Gemini API (batch of 8 pages)
        ✓ Found 27 vehicles
    ...
✓ Completed in 8m 31s

[4/4] Final Processing & Deduplication
✓ Completed in 2.1s

================================================================================
                          COMPLETION SUMMARY
================================================================================

Timing Breakdown:
  Stage 1:.......................... 2m 15s (15.3%)
  Stage 2:.......................... 3m 42s (25.4%)
  Stage 3:.......................... 8m 31s (58.1%)
  Stage 4:..........................   2.1s (1.2%)
  ─────────────────────────────────────────────
  Total:........................... 14m 30s

✓ Generated inventory with 52 vehicles
ℹ Saved to: /path/to/data/inventory.json

Quick Stats:
  Vehicles: 52
  Avg Price: $20,236
  Price Range: $11,333 - $24,056

ℹ To view the site locally, run:
  cd site && npm run dev
```

### Option 2: Individual Stages

Run stages separately for debugging:

```bash
# Stage 1: Find dealerships
python scripts/stage1_dealerships.py

# Stage 2: Search inventory pages
python scripts/stage2_inventory.py

# Stage 3: Parse with Gemini
python scripts/stage3_parse.py

# Stage 4: Final processing
python scripts/fetch.py
```

### Option 3: Run Frontend Only

If you already have `data/inventory.json`, just run the frontend:

```bash
cd site
npm run dev
```

Then open http://localhost:5173

## Understanding the Output

### Request Log Format

```
→ METHOD STATUS  TIME    SIZE   URL
→ GET    200    1.2s   15.2KB  https://example.com/page
```

- **METHOD**: HTTP method (GET/POST)
- **STATUS**: HTTP status code
  - 🟢 200-299: Success (green)
  - 🟡 300-399: Redirect (yellow)
  - 🔴 400+: Error (red)
- **TIME**: Request duration
  - `< 1s`: Shown in milliseconds (e.g., `450ms`)
  - `≥ 1s`: Shown in seconds (e.g., `1.2s`)
- **SIZE**: Response size (B/KB/MB)
- **URL**: Request URL (truncated if too long)

### Stages Explained

1. **Stage 1 - Finding Dealerships**
   - Searches Google for dealerships in nearby cities
   - Filters out aggregators (cars.com, etc.) and manufacturer sites
   - Caches results for 7 days
   - Output: `data/.cache/stage1_dealerships.json`

2. **Stage 2 - Searching Inventory Pages**
   - Searches each dealership's website for inventory pages
   - Filters out non-inventory pages (about, contact, etc.)
   - Output: `data/.cache/stage2_inventory_pages.json`

3. **Stage 3 - Parsing with Gemini AI**
   - Fetches HTML from inventory pages
   - Processes in batches of 8 pages per Gemini request
   - 4 second delay between batches (rate limiting)
   - Extracts vehicle data (year, make, model, price, etc.)
   - Output: `data/.cache/stage3_vehicles.json`

4. **Stage 4 - Final Processing**
   - Deduplicates vehicles by VIN/hash
   - Calculates finance estimates
   - Tracks price changes
   - Applies filters (year, mileage, makes)
   - Output: `data/inventory.json`

## Troubleshooting

### "Missing required environment variables"

Make sure all API keys are set:
```bash
echo $GOOGLE_API_KEY
echo $GOOGLE_CSE_ID
echo $GEMINI_API_KEY
```

If empty, export them again.

### "Rate limit exceeded"

Google Custom Search has quotas:
- Free tier: 100 queries/day
- Paid tier: Higher limits

If you hit limits:
- Wait 24 hours for reset, or
- Use cached results (delete `.env` file to force cache usage)

### "Gemini API timeout"

If Gemini requests timeout:
- Check your internet connection
- Verify Gemini API key is valid
- Some batches may be too large; the timeout is set to 60s

### "No vehicles found"

Possible reasons:
- Dealership websites have changed structure
- Search filters are too restrictive
- Gemini couldn't parse the HTML

Check intermediate cache files in `data/.cache/` to debug.

## Performance Tips

### Faster Runs

1. **Use cached dealerships**: Don't delete `data/.cache/stage1_dealerships.json`
2. **Reduce search scope**:
   - Fewer cities in `google_search.py`
   - Fewer makes in config
3. **Limit inventory pages**: Edit stages to process fewer pages

### Cost Optimization

1. **Google Search API**: Each search = 1 query
   - Stage 1: ~30-50 queries
   - Stage 2: ~100-150 queries
   - Total: ~150-200 queries per full run

2. **Gemini API**: Pricing per token
   - Stage 3: ~20-30 requests
   - Each request: 10k-50k input tokens, 1k-8k output tokens
   - Estimate: $0.50-$2.00 per full run (varies by results)

## File Structure

```
buy-a-car/
├── run_local.py              # Main local runner script
├── config/
│   └── app.config.json       # Search configuration
├── scripts/
│   ├── stage1_dealerships.py # Stage 1: Find dealers
│   ├── stage2_inventory.py   # Stage 2: Search inventory
│   ├── stage3_parse.py       # Stage 3: Parse with AI
│   ├── fetch.py              # Stage 4: Final processing
│   └── sources/
│       └── google_search.py  # Google Search integration
├── data/
│   ├── inventory.json        # Final inventory (public)
│   ├── history.json          # Price history (public)
│   └── .cache/               # Intermediate results
│       ├── stage1_dealerships.json
│       ├── stage2_inventory_pages.json
│       └── stage3_vehicles.json
└── site/
    ├── src/                  # React frontend
    └── dist/                 # Built frontend
```

## Next Steps

After generating inventory locally:

1. **View the site**:
   ```bash
   cd site && npm run dev
   ```

2. **Build for production**:
   ```bash
   cd site && npm run build
   ```

3. **Deploy to GitHub Pages**:
   - Push changes to GitHub
   - GitHub Actions will automatically deploy

## Need Help?

- Report issues: https://github.com/t-boris/buy-a-car/issues
- Check logs in terminal for detailed error messages
- Review intermediate cache files in `data/.cache/`
