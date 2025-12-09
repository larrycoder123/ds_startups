# Austrian Startups Analysis & Prediction

A data science project that builds a comprehensive dataset of Austrian startups through web scraping and enrichment, then applies machine learning to predict funding success patterns.

## Overview

This project aggregates startup data from multiple public sources, enriches it with web-scraped information and AI-generated features, and builds a predictive model to identify startups likely to receive funding. The analysis explores sector and regional patterns across the Austrian startup ecosystem.

**Dataset**: 389 Austrian startups with 22 features including location, sector, business model, social media presence, company age, and textual quality metrics.

## Key Features

- **Web Scraping Pipeline**: Automated data collection from EU-Startups directory
- **Data Enrichment**: Enhanced with social media validation, website analysis, and OpenAI-powered text quality scoring
- **Exploratory Analysis**: Sector distribution, regional patterns, and funding landscape insights  
- **Predictive Modeling**: Random Forest classifier with SMOTE to predict funding status

## Tech Stack

**Languages & Core Libraries**
- Python, Pandas, NumPy

**Web Scraping & APIs**
- BeautifulSoup, Requests
- OpenAI API (GPT-4o-mini for text analysis)

**Machine Learning**
- scikit-learn (Random Forest, preprocessing, model selection)
- imbalanced-learn (SMOTE)
- Matplotlib (visualizations)

**Development**
- Jupyter Notebooks

## Project Structure

```
code/
├── analysis.ipynb           # Exploratory data analysis
├── prediction.ipynb         # ML model training & evaluation
├── data/
│   └── startups_final.csv   # Final processed dataset
└── webscraping/
    ├── 1_scrape_startupeu.ipynb      # Initial data collection
    ├── 2_clean_startupeu.ipynb       # Data cleaning
    ├── 3_X_enrich_*.ipynb            # Feature enrichment steps
    └── 4_final_processing_*.ipynb    # Final data preparation
```

## Key Findings

- **Class Imbalance**: Significant disparity between funded (~15%) and non-funded startups
- **Regional Patterns**: Vienna dominates the startup ecosystem, followed by Styria and Upper Austria
- **Sector Distribution**: Software & Analytics is the most common category
- **Predictive Features**: Social media presence, text quality scores, and company age show importance in funding prediction
- **Model Performance**: Random Forest with SMOTE balancing achieved macro F1-score optimization through hyperparameter tuning

## Documentation

See `project_overview.pdf` for detailed methodology, findings, and visualizations.
