# Predicting Used-Car Asking Prices in India: Age, Kilometres, Brand and Depreciation

**Business Analytics (23CSE452) – Individual Case Study** · Akshay KS · CB.SC.U4CSE23104 · CSE-B

## Problem statement
Dealers and private sellers set used-car asking prices from one nearby advertisement or a flat yearly markdown. Price too high and the car sits unsold; too low and margin is lost. This case study tests whether public listing attributes (age, kilometres, fuel, transmission, body type, ownership, brand, city) give a more defensible price, and measures depreciation as *residual value* against current new-car prices.

## Objectives
1. Analyse how age and kilometres driven relate to used-car asking price, including residual value where a current new-car mid price can be matched.
2. Examine the effect of brand, fuel type, transmission, body type, ownership and city on asking price.
3. Build and compare Linear Regression, Decision Tree and Random Forest models on the same split using R², MAE and RMSE.

## Data collection
- **Source:** public [Cars24](https://www.cars24.com/buy-used-cars/) used-car search pages. No Kaggle/UCI/GitHub dataset was used.
- **Method:** a Python (`requests`) scraper reads the listing JSON embedded in each page, from a fixed list of 745 start URLs (base, city and brand pages plus city×brand, city×fuel and city×body pages). Each page shows about 20 cards and the scraper does not paginate. Duplicates are removed on the listing's `appointment_id`.
- **Size:** 6,693 unique listings, 6,629 after cleaning (22 brands, 212 models, 93 cities).
- **Apify check:** the same 745 URLs were also run through Apify Web Scraper (12,560 rows stored, 6,766 unique ids). It is a second collection and is not used for modelling.
- **New-car reference:** 311 current nameplates scraped from public [CarDekho](https://www.cardekho.com/newcars) pages, used only to compute residual value.
- **Privacy:** the free-text `locality` field (seller-typed addresses, occasionally phone numbers) was removed from every data file. Only `city` is kept.

| File | Contents |
|---|---|
| `data/raw/used_cars_raw.csv` | Python scrape (6,693 rows, 18 columns) |
| `data/raw/used_cars_apify_full.csv` | Apify full export (12,560 rows) |
| `data/reference/new_car_mid_prices.csv` | CarDekho new-car price reference (311 nameplates) |
| `data/processed/used_cars.csv` | Cleaned analysis dataset (6,629 rows, 23 columns) |

## Analytics methods
Cleaning (dedupe, age 0–25 years, 200–250,000 km, ₹0.4–150 lakh, passenger cars only), exploratory analysis, then supervised regression on `asking_price_lakh` with Linear Regression, Decision Tree (`max_depth=8`) and Random Forest (280 trees, `max_depth=14`). Predictors are age, kilometres, ownership, fuel, transmission, body type, brand and city. Scoring uses an 80/20 split (`random_state=42`) plus 5-fold cross-validation on the training split. Residual value is a lookup only and is never a model input. The report also compares the work with four published used-car pricing studies.

## Key results
| Model | Test R² | MAE (lakh) | RMSE (lakh) | 5-fold CV R² |
|---|---|---|---|---|
| Linear Regression | 0.668 | 1.827 | 2.914 | 0.609 |
| Decision Tree | 0.716 | 1.662 | 2.695 | 0.664 |
| **Random Forest** | **0.800** | **1.379** | **2.262** | **0.777** |

- Age is the strongest pricing signal (r = −0.46); kilometres are weaker (r = −0.23). A young high-kilometre car (median ₹7.96 lakh) still lists well above an old low-kilometre one (₹2.90 lakh), so a single yearly markdown is a weak rule.
- Matched listings hold a median 58% of the current new mid-price. Kia sits near 83% and Mercedes near 16%: luxury keeps a rupee premium but not a percentage one.
- Random Forest errors are tight for everyday cars but about 3.5× wider above ₹15 lakh (residual std 5.42 vs 1.56 lakh), with mean residual near zero. Treat its estimate as the centre of a negotiation band, and be more cautious on luxury listings.
- Limits: asking price is not the final sale price, condition and service history are unobserved, and the data is a single snapshot.

## Repository contents
```
README.md                 this file
analysis.ipynb            full analysis with outputs (cleaning, EDA, models, evaluation)
Case_Study_Report.pdf     final report
data/                     raw, reference and cleaned datasets (above)
```

## Reproduce
```bash
pip install pandas numpy scikit-learn matplotlib seaborn jupyter
jupyter notebook analysis.ipynb   # run from the repository root
```
The notebook reads from `data/` and recreates a `figures/` folder when run.

## References
1. Cui, B., Ye, Z., Zhao, H., Renqing, Z., Meng, L. and Yang, Y. (2022) 'Used Car Price Prediction Based on the Iterative Framework of XGBoost+LightGBM', *Electronics*, 11(18), 2932. doi: 10.3390/electronics11182932.
2. Bhatt, N.S., Pandey, T.N., Reddy, S.R., Jayasurya, B., Dash, B.B. and Patra, S.S. (2023) 'An Emperical Analysis of Machine Learning Algorithms for Used Car Price Prediction System', *2023 Global Conference on Information Technologies and Communications (GCITC)*, IEEE, pp. 1–5. doi: 10.1109/GCITC60406.2023.10426270.
3. Sanke, M., Naik, Y., Pillai, G., Ghode, R. and Lamani, A. (2024) 'Machine Learning Powered Chatbot for Prediction of Used Car Price', *International Journal of Computer Applications*, 186(37), pp. 8–13. doi: 10.5120/ijca2024923900.
4. Li, C. (2024) 'Machine Learning-Based Models for Accurate Car Prices Prediction', *Highlights in Business, Economics and Management*, 40, pp. 416–421. doi: 10.54097/9zcpv779.
5. Cars24 (2026) *Buy used cars*. https://www.cars24.com/buy-used-cars/ (accessed 18 September 2026).
6. CarDekho (2026) *New cars*. https://www.cardekho.com/newcars (accessed 18 September 2026).
7. Apify (2026) *Web Scraper*. https://apify.com/apify/web-scraper (accessed 18 September 2026).
8. Pedregosa, F. et al. (2011) 'Scikit-learn: Machine Learning in Python', *Journal of Machine Learning Research*, 12, pp. 2825–2830.
