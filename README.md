# Cinema Audience Forecasting

## Live Report

The complete interactive project report with all analysis, visualizations, and results is available here:  
**[View Detailed Project Report](https://22f3001706.github.io/Cinema-Audience-Forecasting-challenge/report.html)**

## Project Overview

This repository contains a solution for a cinema audience forecasting competition. The goal was to predict daily audience counts for movie theaters over a two-month period (March to April 2024) using historical data from January 2023 to February 2024. The solution employs time series analysis and gradient boosting techniques to generate accurate forecasts.

## Dataset Description

The competition provided multiple data files from two booking platforms: BookNow and CinePOS. The data spans from January 1, 2023 to February 28, 2024.

### Input Files

| File Name | Description |
|-----------|-------------|
| movie_theater_id_relation.csv | Mapping between BookNow and CinePOS theater IDs |
| date_info.csv | Calendar information with show dates and days of week |
| sample_submission.csv | Submission template with ID format and zero-filled predictions |
| booknow_theaters.csv | Theater metadata for BookNow platform |
| cinePOS_booking.csv | Booking records from CinePOS platform |
| cinePOS_theaters.csv | Theater metadata for CinePOS platform |
| booknow_visits.csv | Historical audience counts for BookNow theaters |
| booknow_booking.csv | Booking records from BookNow platform |

### Data Shape Summary

- movie_theater_id_relation: (150, 2)
- date_info: (547, 2)
- booknow_theaters: (829, 5)
- cinePOS_booking: (1,641,966, 4)
- cinePOS_theaters: (4,690, 5)
- booknow_visits: (214,046, 3)
- booknow_booking: (68,336, 4)
- sample_submission: (38,062, 2)

## Exploratory Data Analysis

### Key Findings

1. Stationarity: The audience count time series is stationary based on the Augmented Dickey-Fuller test (p-value = 0.0).

2. Seasonal Patterns: Clear weekly patterns exist with a 7-day cycle visible in both additive and multiplicative decomposition.

3. Date Range:
   - Training data: 2023-01-01 to 2024-02-28
   - Prediction period: 2024-03-01 to 2024-04-22

4. Data Quality:
   - booknow_theaters contains 515 missing book_theater_id values
   - cinePOS_theaters has 3,861 missing latitude and longitude values
   - Duplicate records exist in multiple datasets and were removed

5. Audience Distribution:
   - Mean audience: 41.62
   - Median audience: 34.00
   - Maximum audience: 1,350
   - Outliers identified beyond 1.5 IQR: 5,589 records

### Time Series Decomposition

Both additive and multiplicative decomposition models were applied with a period of 7 days. The analysis revealed:

- Clear trend component showing overall patterns over time
- Regular seasonal fluctuations around the trend
- Residual noise that becomes more pronounced during high-activity periods

## Methodology

### Feature Engineering

Two primary feature engineering approaches were implemented:

1. Temporal Features:
   - Day of week, month, quarter, week of year, day of year
   - Weekend indicator (binary)
   - Month start and month end indicators
   - Sine and cosine transformations of day of year for cyclical patterns

2. Lag and Rolling Features:
   - Lag features: 1, 2, 7, 14 days
   - Rolling means: 7-day and 14-day windows

### Model Development

Three modeling approaches were evaluated:

#### SARIMA (Seasonal ARIMA)
- Order: (1,1,1)
- Seasonal order: (2,1,2,7)
- Strengths: Captures seasonality natively
- Limitations: Less effective with complex feature interactions

#### XGBoost
- Parameters after tuning:
  - n_estimators: 800
  - learning_rate: 0.02
  - max_depth: 5
  - subsample: 0.8
  - colsample_bytree: 1.0
- Validation performance:
  - MAE: 14.41
  - RMSE: 20.66

#### LightGBM
- Parameters:
  - n_estimators: 1000
  - learning_rate: 0.02
  - max_depth: -1
  - num_leaves: 70
  - subsample: 0.8
  - colsample_bytree: 0.8
- Validation performance:
  - MAE: 14.62
  - RMSE: 20.90

### Final Model Selection

XGBoost was selected as the final model due to:
- Superior handling of engineered features
- Better validation metrics compared to LightGBM
- More robust performance than SARIMA for this specific forecasting task

### Prediction Strategy

Recursive forecasting was implemented for each theater:
1. Combine historical data with future prediction dates
2. Train model on complete historical dataset
3. Generate predictions sequentially, updating lag features after each prediction
4. Round negative predictions to zero (audience count cannot be negative)

## Results

The final submission achieved a validation MAE of 14.41 and RMSE of 20.66 on the validation period (2024-01-15 to 2024-02-28).

### Sample Predictions

| ID | Predicted Audience |
|----|-------------------|
| book_00001_2024-03-01 | 31.16 |
| book_00001_2024-03-02 | 41.12 |
| book_00001_2024-03-03 | 45.24 |
| book_00001_2024-03-04 | 50.05 |
| book_00001_2024-03-06 | 32.26 |

## Key Insights from Analysis

1. The time series data is stationary, making it suitable for standard time series modeling approaches.

2. Clear weekly seasonality exists in audience attendance patterns.

3. The mapping between BookNow and CinePOS theaters covers 150 theaters, allowing cross-platform feature integration.

4. Missing values in latitude/longitude data require careful handling for any spatial analysis.

5. The majority of bookings occur on the same day as the show (20,433 out of 68,336 records).

6. Sunday accounts for approximately 13.32% of all bookings.

## Technical Requirements

The solution requires the following Python packages:

- numpy
- pandas
- matplotlib
- seaborn
- statsmodels
- xgboost
- lightgbm
- scikit-learn


