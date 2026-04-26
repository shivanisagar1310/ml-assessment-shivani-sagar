## 1. Problem Formulation

(a) Machine Learning Problem Definition

Target Variable:'items_sold' (number of items sold per store per month)

Candidate Input Features:
  * Promotion-related: 'promotion_type'
  * Store characteristics: 'store_size', 'location_type', 'competition_density'
  * Temporal features: 'month', 'day_of_week', 'is_weekend', 'is_festival', 'is_month_end'
  * Store identifier: 'store_id' (to capture store-level patterns)

Type of ML Problem: Supervised Regression

Justification:
  The objective is to predict a continuous numeric outcome (items sold) based on historical labeled data. Since past observations include both input features and corresponding sales outcomes, this is a supervised learning setup. Regression is appropriate because the target variable is continuous and ordered.

-----------------------------------------------------------------------------------------

(b) Why “Items Sold” is Better than Revenue

Using items sold (sales volume) is more reliable than total revenue because:

* Revenue is influenced by price variations, discounts, and promotion mechanics, which can impact the true demand
* A promotion like flat discounts may increase volume but reduce revenue per item
* Different stores may have different pricing structures, making revenue less comparable across locations

Key Principle:
The target variable should reflect the true business objective being optimized, not a derived or confounded metric.
In this case, the goal is to understand promotion effectiveness on demand, and items sold is a direct measure of customer response, whereas revenue mixes demand with pricing effects.

--------------------------------------------------------------

(c) Alternative to a Single Global Model

A single global model assumes that all stores respond similarly to promotions, which is unrealistic given differences in:

* Customer demographics
* Location (urban vs rural)
* Competition intensity

Proposed Approach: Segment-Based or Hierarchical Modeling

* Build separate models for different store segments (e.g., urban, semi-urban, rural), OR
* Use a hierarchical (multi-level) model that captures both global trends and store-specific variations

Justification:
This approach:
* Captures heterogeneity in customer behavior
* Improves prediction accuracy by tailoring models to store characteristics
* Enables more precise, localized promotion strategies

This leads to better decision-making compared to a one-size-fits-all global model.

------------------------------------------------------------------------------

## B2. Data and EDA Strategy

(a) Data Integration and Dataset Design

The raw data is spread across four tables:
* Transactions (sales records)
* Store attributes (store characteristics)
* Promotion details (type of promotion applied)
* Calendar (weekend and festival indicators)

Joining Strategy:
* Join transactions and store attributes using 'store_id'
* Join transactions and promotion details using 'promotion_id' or promotion identifier
* Join transactions and calendar using 'transaction_date'

Final Dataset Grain:

* One row = one store per day (or per transaction_date)

Aggregations:
Before modeling, aggregate data at the store-date level:

* Total 'items_sold' (target variable)
* Average or total transaction-level metrics (if available)
* Promotion type applied on that date
* Calendar features (weekend, festival)

Outcome:A clean, structured dataset where each row represents store-level performance for a given date, suitable for supervised learning.

----------------------------------------------------------------------

(b) Exploratory Data Analysis (EDA)

1. Sales Distribution (Histogram / Boxplot)
* What to check:** Distribution of 'items_sold', presence of skewness or outliers
* Impact: May require log transformation or outlier handling

2. Sales by Promotion Type (Bar Chart)

* What to check: Average items sold across different promotions
* Impact: Helps identify high-performing promotions and guides feature importance

3. Time Series Trend (Line Plot)

* What to check: Sales trends over time (monthly/weekly patterns)
* Impact: Justifies creation of temporal features (month, weekend, seasonality)

4. Correlation Analysis (Heatmap)

* What to check: Relationships between numerical variables and sales
* Impact: Identifies key drivers and removes redundant features

5. Sales by Store Segment (Boxplot by location/store size)

* What to check: Variation in sales across store types
* Impact: Supports segmentation strategy or inclusion of interaction features

---------------------------------------------------------------------------------

(c) Handling Promotion Imbalance

Observation:
80% of transactions have no promotion, leading to class imbalance.

Impact on Model:

* Model may learn to bias toward “no promotion” scenarios
* Reduced ability to accurately estimate promotion effects
* Underestimation of uplift from promotional activities

Mitigation Strategies:
* Ensure promotion-related features are properly encoded
* Use stratified sampling or weighting techniques
* Evaluate model performance separately on promotion vs non-promotion cases
* Consider creating a binary feature (promotion vs no promotion)

Conclusion: Addressing imbalance is critical to ensure the model captures the true impact of promotions rather than defaulting to the majority class.

--------------------------------------------------------------------------------------------------

## B3. Model Evaluation and Deployment

(a) Train-Test Split and Evaluation Metrics

Train-Test Split Strategy:
Use a time-based split:
  * Train on the first ~2.5 years of data
  * Test on the most recent ~6 months
This ensures the model is trained on past data and evaluated on future data.

Why Random Split is Inappropriate:
* Breaks temporal order → introduces data leakage
* Model may learn patterns from the future that wouldn’t be available in real-world predictions
* Leads to overly optimistic and unrealistic performance

Evaluation Metrics:

1. RMSE (Root Mean Squared Error)
* Measures overall prediction error
* Penalizes large errors more heavily
* Business meaning: Large mistakes in demand prediction (e.g., underestimating peak sales) are costly

2. MAE (Mean Absolute Error)
* Measures average absolute prediction error
* Easier to interpret in business terms
* *Business meaning: On average, how many units the prediction is off

3. (Optional) R² Score
* Measures proportion of variance explained
* Business meaning: How well the model captures overall demand patterns

Interpretation:
* Lower RMSE and MAE indicate better model performance
* MAE is more interpretable, while RMSE highlights risk from large errors

---------------------------------------------------------------------------------

(b) Explaining Model Recommendations Using Feature Importance

The model recommends:
* Loyalty Points Bonus (December)
* Flat Discount (March) for the same store

Investigation Approach:
1. Feature Importance Analysis
   * Identify top features influencing predictions (e.g., month, festival, promotion type)

2. Month-Based Differences
   * December likely includes:
     * Festival/holiday effects
     * Higher customer spending behavior
   * March may represent:
     * Regular demand period
     * More price-sensitive customers

3. Compare Feature Values
   * Examine input features for both months:
     * 'is_festival', 'month', 'weekend', etc.

---

Communication to Marketing Team:
* In December, customers are less price-sensitive and more responsive to loyalty incentives due to festive demand.
* In March, demand is lower and more price-driven, making discounts more effective.

Key Insight:
The model is not recommending randomly—it is adapting to seasonality and customer behavior patterns.

-----------------------------------------------------------------------------------------------------

(c) Deployment Strategy

1. Model Saving
* Save the trained pipeline using:
  * 'joblib' or 'pickle'
* Include: Preprocessing steps, Trained model

2. Monthly Prediction Workflow

At the start of each month:
1. Collect latest data:
   * Store attributes
   * Promotion options
   * Calendar features

2. Apply same preprocessing pipeline:
   * Feature engineering
   * Encoding
   * Scaling

3. Generate predictions:
   * Predict expected `items_sold` for each promotion option
   * Select promotion with highest predicted sales


3. Monitoring and Maintenance

Performance Monitoring:
* Track actual vs predicted sales monthly
* Monitor RMSE and MAE over time
 Drift Detection:

* Identify changes in:
  * Customer behavior
  * Promotion effectiveness
  * Data distribution

Retraining Trigger:
* Significant drop in performance metrics
* Periodic retraining (e.g., quarterly)

Conclusion:
A robust deployment pipeline ensures that:

* Predictions are consistent and automated
* Business teams receive timely recommendations
* Model performance is continuously monitored and improved
