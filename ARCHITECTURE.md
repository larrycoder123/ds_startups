# Architecture Overview

This document provides a comprehensive overview of the Austrian Startups Analysis project structure, data flow, and component relationships.

## 📁 Project Structure

```
ds_startups/
├── README.md                    # Project overview
├── ARCHITECTURE.md              # This file
├── project_overview.pdf         # Detailed project documentation
├── presentation.pdf             # Project presentation
├── .gitignore
├── .venv/                       # Python virtual environment
│
└── code/
    ├── analysis.ipynb           # Exploratory data analysis (minimal)
    ├── prediction.ipynb         # ML model training & evaluation
    ├── data/
    │   └── startups_final.csv   # Final clean dataset for modeling
    │
    └── webscraping/
        ├── 1_scrape_startupeu.ipynb
        ├── 2_clean_startupeu.ipynb
        ├── 3_1_enrich_startupeu.ipynb
        ├── 3_2_enrich_startupeu.ipynb
        ├── 3_3_enrich_startupeu.ipynb
        ├── 3_4_enrich_startup.ipynb
        ├── 3_5_enrich_startup.ipynb
        ├── 4_final_processing_startupeu.ipynb
        ├── adding_variables_and_rankings.ipynb    # Additional feature engineering
        ├── graphs_and_analysis_adjusted.ipynb     # Visualization & EDA
        ├── data/                                  # Intermediate data files
        │   ├── eustartup_listings.csv             # Raw scraped data
        │   ├── eustartup_listings_cleaned.csv
        │   ├── eustartup_listings_enriched_1.csv
        │   ├── eustartup_listings_enriched_2.csv
        │   ├── eustartup_listings_enriched_3.csv
        │   ├── eustartup_listings_enriched_4.csv
        │   ├── eustartup_listings_enriched_5.csv
        │   ├── eustartup_listings_final.csv
        │   ├── startup_df.csv                     # Dataset with status labels
        │   └── ...                                # Helper/temp files
        └── old/                                   # Deprecated notebooks
```

---

## 🔄 Data Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA COLLECTION                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1_scrape_startupeu.ipynb                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Web scraping from EU-Startups directory (66 pages)                │  │
│   │  • BeautifulSoup + Requests                                          │  │
│   │  • Extracts: name, link, category, city, tags, founded year          │  │
│   │  → Output: eustartup_listings.csv                                    │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                           DATA CLEANING                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   2_clean_startupeu.ipynb                                                    │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Remove duplicates (by name, website)                              │  │
│   │  • Process long descriptions                                         │  │
│   │  • Standardize city/region names                                     │  │
│   │  • Clean URL formats                                                 │  │
│   │  → Output: eustartup_listings_cleaned.csv                            │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                        ENRICHMENT PIPELINE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   3_1_enrich_startupeu.ipynb  (OpenAI API)                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • GPT-4o-mini analysis of business descriptions                     │  │
│   │  • Extracts: writing_score, clarity_score, innovativeness_score,     │  │
│   │    market_readiness_score, founder_signal_score, sentiment_score,    │  │
│   │    traction_score, word_count, jargon_density, top_3_keywords,       │  │
│   │    business_model (B2B/B2C/marketplace/etc.)                         │  │
│   │  → Output: eustartup_listings_enriched_1.csv                         │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
│   3_2_enrich_startupeu.ipynb  (Website Validation)                           │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Clean and validate URLs                                           │  │
│   │  • HTTP HEAD requests to check website availability                  │  │
│   │  • Resolve redirects, handle timeouts                                │  │
│   │  → Output: eustartup_listings_enriched_2.csv                         │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
│   3_3_enrich_startupeu.ipynb  (Social Media Discovery)                       │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Clean company names (remove GmbH, AG suffixes)                    │  │
│   │  • Search API requests for:                                          │  │
│   │    - LinkedIn company profiles                                       │  │
│   │    - Instagram handles                                               │  │
│   │    - X (Twitter) handles                                             │  │
│   │  → Output: eustartup_listings_enriched_3.csv                         │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
│   3_4_enrich_startup.ipynb  (News/Headlines - Serper API)                    │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Google News search via Serper.dev API                             │  │
│   │  • Filter headlines by funding/startup keywords                      │  │
│   │  • Extract funding amounts, acquisition mentions, bankruptcy signals │  │
│   │  → Output: eustartup_listings_enriched_4.csv                         │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
│   3_5_enrich_startup.ipynb  (Social Media Validation)                        │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Validate LinkedIn URLs (AT domain vs global)                      │  │
│   │  • Create boolean flags: linkedin_valid, instagram_valid, x_valid    │  │
│   │  • Clean and standardize handles                                     │  │
│   │  → Output: eustartup_listings_enriched_5.csv                         │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                        FINAL PROCESSING                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   4_final_processing_startupeu.ipynb                                         │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Column selection & renaming                                       │  │
│   │  • Remove invalid handles where validation failed                    │  │
│   │  • Create target variable: status (funding/no_funding/inactive)      │  │
│   │  • Final data quality checks                                         │  │
│   │  → Output: eustartup_listings_final.csv, startup_df.csv              │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                    ↓                                         │
│   adding_variables_and_rankings.ipynb  (Additional Features)                 │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Add economic variables                                            │  │
│   │  • Create rankings                                                   │  │
│   │  → Output: eustartup_listings_adjusted_with_economic_vars.csv        │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                     ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ANALYSIS & MODELING                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   graphs_and_analysis_adjusted.ipynb                                         │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Exploratory Data Analysis                                         │  │
│   │  • Visualization: sector distribution, regional patterns             │  │
│   │  • Statistical summaries                                             │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   prediction.ipynb                                                           │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  • Load: startup_df.csv → code/data/startups_final.csv               │  │
│   │  • Feature selection (22 features)                                   │  │
│   │  • Preprocessing: imputation, one-hot encoding                       │  │
│   │  • SMOTE for class imbalance                                         │  │
│   │  • Random Forest with RandomizedSearchCV                             │  │
│   │  • Evaluation: confusion matrix, F1-macro, feature importances       │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🗃️ Data Schema

### Final Dataset (`startups_final.csv`)

| Column | Type | Description |
|--------|------|-------------|
| `name` | str | Startup name (cleaned) |
| `link_startupeu` | str | EU-Startups directory URL |
| `link_website` | str | Company website (validated) |
| `city` | str | City location |
| `region` | str | Austrian region (Vienna, Styria, etc.) |
| `category` | str | Industry sector |
| `top_3_keywords` | list | AI-extracted keywords |
| `business_model` | str | B2B, B2C, B2B2C, marketplace, platform, other |
| `linkedin_global_profile` | bool | Has global LinkedIn (not AT-specific) |
| `linkedin_valid` | bool | Valid LinkedIn presence |
| `instagram_valid` | bool | Valid Instagram presence |
| `x_valid` | bool | Valid X/Twitter presence |
| `age` | int | Years since founding |
| `total_funding_listing` | str | Funding range from listing |
| `headline_non_financial_count` | int | Non-funding related headlines |
| `description_writing_score` | int | Writing quality (1-10) |
| `description_market_readiness_score` | int | Market readiness (1-10) |
| `description_founder_signal_score` | int | Founder credibility (1-10) |
| `description_word_count` | int | Description length |
| `description_jargon_density` | float | Jargon ratio (0-1) |
| `description_numeric_evidence_count` | int | Numeric mentions count |
| `status` | str | **Target**: funding, no_funding, inactive |

---

## 🔧 External Dependencies & APIs

| Service | Purpose | Notebook |
|---------|---------|----------|
| **OpenAI API** | GPT-4o-mini for text analysis | `3_1_enrich_startupeu.ipynb` |
| **Serper.dev API** | Google News search | `3_4_enrich_startup.ipynb` |
| **HTTP Requests** | Website validation, social media discovery | `3_2`, `3_3`, `3_5` |

### Required Environment Variables (.env)
```
OPENAI_API_KEY=sk-...
SERPER_DEV_KEY=...
```

---

## 🐍 Tech Stack

### Core Libraries
- **pandas** - Data manipulation
- **numpy** - Numerical operations
- **requests** - HTTP requests
- **beautifulsoup4** - HTML parsing

### Machine Learning
- **scikit-learn** - ML pipeline, Random Forest, preprocessing
- **imbalanced-learn** - SMOTE for class imbalance

### Visualization
- **matplotlib** - Plotting

### APIs & Utilities
- **openai** - GPT API client
- **python-dotenv** - Environment variable management
- **tqdm** - Progress bars

---

## ⚠️ Known Issues & Technical Debt

### Structural Issues
1. **Fragmented pipeline**: 8+ sequential notebooks make reproduction difficult
2. **No modular code**: All logic embedded in notebooks, no reusable `.py` modules
3. **Duplicate analysis notebooks**: `analysis.ipynb` (empty), `graphs_and_analysis_adjusted.ipynb`
4. **Scattered data files**: 14+ CSV files in `webscraping/data/` with unclear lineage
5. **Hardcoded paths**: Relative paths vary (`./data/`, `./webscraping/data/`, `./data_new/`)

### Code Quality
1. **No error handling**: API calls lack retry logic or graceful failure
2. **No logging**: Silent failures in enrichment steps
3. **No tests**: Zero test coverage
4. **No type hints**: Function signatures lack typing
5. **Repeated boilerplate**: Same pandas display options set in every notebook

### Data Issues
1. **Intermediate files committed**: Large CSVs tracked in git
2. **No data validation**: Schema not enforced between pipeline steps
3. **Manual interventions**: Some notebooks require manual cell-by-cell execution

---

## 🚀 Recommended Improvements

### High Priority
- [ ] Consolidate notebooks into modular Python scripts
- [ ] Create single pipeline entry point (e.g., `make run` or `dvc repro`)
- [ ] Add `requirements.txt` or `pyproject.toml` with pinned versions
- [ ] Implement logging throughout pipeline
- [ ] Add data validation between steps (pandera/pydantic)

### Medium Priority
- [ ] Move intermediate data to `.gitignore`, use DVC for data versioning
- [ ] Add retry logic for API calls
- [ ] Create unified config file for paths/API keys
- [ ] Add docstrings and type hints

### Low Priority
- [ ] Add unit tests for data transformations
- [ ] Create CI/CD pipeline for reproducibility checks
- [ ] Add pre-commit hooks (black, isort, flake8)

---

## 📊 Dataset Statistics

- **Total startups**: ~389 (after deduplication)
- **Features**: 22 selected for modeling
- **Target classes**: 
  - `funding` (~15%)
  - `no_funding` (~70%)
  - `inactive` (~15%)
- **Regions covered**: 9 Austrian regions
- **Categories**: 15+ industry sectors
