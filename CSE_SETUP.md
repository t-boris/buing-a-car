# Google Custom Search Engine (CSE) Setup

If you're getting 400 errors when searching specific dealer sites, your CSE might need reconfiguration.

## Problem

```
✗ Search error for 'inventory': Client error '400 Bad Request'
```

This happens when your CSE is **not configured to search the entire web**.

## Solution: Create a New CSE

### 1. Go to Google Programmable Search Engine

https://programmablesearchengine.google.com/

### 2. Create New Search Engine

Click "Add" or "Create"

### 3. Configure Settings

**Sites to search:**
- Select **"Search the entire web"**
- ⚠️ Do NOT select "Search only included sites"

**Search features:**
- Enable "Image search" (optional)
- Enable "SafeSearch" (optional)

### 4. Get Your CSE ID

After creation:
1. Click "Edit" on your search engine
2. Find "Search engine ID" in the Overview section
3. Copy the ID (looks like: `60da5e45aaeb04134`)

### 5. Update secrets.md

```bash
export GOOGLE_CSE_ID='your-new-cse-id-here'
```

### 6. Test

```bash
source secrets.md
python run_local.py
```

## Verification

Your CSE is configured correctly if:
- ✅ Stage 1 finds dealerships
- ✅ Stage 2 finds inventory pages (not all 0)
- ✅ No 400 errors in logs

## Alternative: Use Existing CSE

If you must use a CSE restricted to specific sites:

1. Add all dealer domains to "Sites to search" in CSE settings
2. This is impractical with 100+ dealers
3. **Recommended:** Use "Search the entire web" instead

## Why This Happens

Google Custom Search has two modes:

**Mode 1: Search the entire web** ✅
- Supports `site:` operator
- Can search any website
- Best for AutoFinder

**Mode 2: Search only included sites** ❌
- Limited to pre-configured sites
- `site:` operator may conflict
- Causes 400 errors

## Current Workaround

The code now has fallback logic:
1. Try site-specific search (`site:domain.com`)
2. If that fails (400), try general search with domain name
3. Filter results to only include target domain

This works but uses more API quota. **Best solution:** Configure CSE to "Search the entire web"

## API Quota

- Free tier: 100 queries/day
- With "Search the entire web": Each run ~150-200 queries
- Upgrade to paid tier if needed

## Need Help?

Check error messages with verbose mode:

```bash
AUTOFINDER_VERBOSE=1 python run_local.py
```

Look for detailed error messages like:
```
✗ Google API 400 error: Invalid site restriction
Query: inventory site:example.com
Site: example.com
```
