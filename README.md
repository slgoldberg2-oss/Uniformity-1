# Cook County Uniformity & Sales Analysis

A full-stack web application for Cook County property tax appeal analysis. Pulls live data from the Cook County Open Data Portal to generate:

- **Uniformity Analysis** — finds comparable properties in the same township with lower building assessment per square foot
- **Comparable Sales Analysis** — finds recent warranty deed sales with lower implied building assessment per square foot
- **Property Records PDF** — generates and downloads a merged PDF of Assessor property records for the subject and all comparables

---

## Tech Stack

- **Backend**: Node.js + Express (proxy, analysis logic, PDF generation via Puppeteer + pdf-lib)
- **Frontend**: React (Create React App)
- **Data**: Cook County Open Data Portal OData API

---

## Local Development

### Prerequisites
- Node.js 18+
- npm

### Setup

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/cook-county-analysis.git
cd cook-county-analysis

# Install all dependencies
npm run install:all

# Run both server and client concurrently
npm run dev
```

- **Server** runs on `http://localhost:3001`
- **Client** runs on `http://localhost:3000` (proxies API requests to server)

---

## Deploy to Railway

### Option 1: Deploy from GitHub (recommended)

1. Push this repo to GitHub
2. Go to [railway.app](https://railway.app) and click **New Project**
3. Select **Deploy from GitHub repo**
4. Choose this repository
5. Railway will auto-detect the `railway.json` and `nixpacks.toml` config
6. Click **Deploy** — Railway handles everything

> **Note**: No environment variables are required. The `PORT` variable is set automatically by Railway.

### Option 2: Railway CLI

```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

---

## Project Structure

```
cook-county-analysis/
├── server/
│   └── index.js          # Express server, API proxy, PDF generation
├── client/
│   ├── public/
│   │   └── index.html
│   └── src/
│       ├── index.js
│       └── App.js        # React frontend
├── package.json          # Root (server deps + build scripts)
├── railway.json          # Railway deployment config
├── nixpacks.toml         # Build config (includes Chromium for Puppeteer)
├── .gitignore
└── README.md
```

---

## Data Sources

| Dataset | URL |
|---|---|
| Property Information | `https://datacatalog.cookcountyil.gov/api/odata/v4/x54s-btds` |
| Assessment Data | `https://datacatalog.cookcountyil.gov/api/odata/v4/uzyt-m557` |
| Sales | `https://datacatalog.cookcountyil.gov/api/odata/v4/wvhk-k5uv` |

---

## How It Works

### Uniformity Analysis
1. Fetches subject property building characteristics and assessment values
2. Searches all properties in the same township for the same tax year
3. Filters to properties with a lower building assessment per square foot
4. Prioritizes comparables by: same block → nearest block → nearest parcel
5. Returns top 4 comparables and total count of lower-assessed properties

### Sales Comparison
1. Searches warranty deed sales within 3 years of the tax year, same township and class
2. Computes implied building assessment: `(Sale Price × 10%) − Land Assessment`
3. Calculates implied building assessment per square foot
4. Returns 4 most recent sales with lower implied bldg assess/sqft than subject

### Property Records PDF
- Uses Puppeteer to load the Cook County Assessor's print page for each PIN
- Captures first 2 pages of each property's record
- Merges all pages into a single downloadable PDF using pdf-lib
