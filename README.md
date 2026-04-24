# bike-rental-prediction
Problem Understanding:
Capital bikeshare needs to anticipate how demands shift across hours, days and seasons.
We have to analyze the bike rental data based on hour, day, weather and season and create a model which predicts the number of bike rentals in a specific hour.

## Exploratory Data Analysis:
a) Plot rental demand vs time and identify visible trends

b) Analyze hourly, daily and seasonal patterns using appropriate visualizations
Observation 1 (Trend): Number of rentals in 2012 is higher than that in 2011. 
Why? Capital Bikeshare was a relatively new program. As people saw the stations and learned how to use the app, adoption naturally grew.
Observation 2 (Hourly): There are massive spikes at 8 AM and 5 PM during workdays. 
Why? Commuters! People are riding bikes to the office/subway in the morning and riding back home in the evening.
The weekend line will look like a single smooth mountain peaking at 1 PM (people sleeping in and going for casual afternoon rides).
Observation 3 (Monthly/Seasonal): Yes, the demand peaks in June and September, with a weird dip in July. 
Why? Washington D.C. weather! June and September are beautiful, but July and August in D.C. are notoriously sweltering and humid. People don't want to show up to work sweating, so rentals drop.

c) Check for autocorrelation - does demand at one hour influence the next?

d) Identify anomalies or irregular spikes and reason about their cause
The dark blue ink suddenly plunges down to almost zero for a couple of days, breaking the entire pattern. 
What happened? Hurricane Sandy hit Washington D.C. on October 29, 2012. The city completely shut down, and nobody was riding bikes.

e) Decompose the series into trend, seasonality, and residual components
1. The Trend Graph 
By isolating the trend component, we can clearly see a steady upward trajectory in overall bike rentals from early 2011 to late 2012. This indicates that the bikeshare program is successfully gaining adoption and growing its user base, regardless of daily weather fluctuations.
2. The Seasonal Graph
The predictable, repeating pattern. The 8 AM spike, the 5 PM spike, repeating exactly the same way every 24 hours.
3. The Residual Graph
The residual plot highlights the anomalies—days where bike demand severely broke the normal patterns. For instance, the massive drop in late October 2012 aligns with Hurricane Sandy. This insight tells us that our predictive model MUST include weather features (like rain, snow, and windspeed); otherwise, it will blindly predict high commuter demand even during a severe storm.

## Feature Engineering:

a) Extract meaningful time-based features from the timestamp-
Columns for Hour (0-23), Day of Week (0-6), and Month (1-12) are already created.

b) Create lag features and rolling statistics-
Lag Features: The ACF plot showed strong correlations at Lag 1 and Lag 24. To give the model "memory," I engineered lag_1 (rentals from the previous hour) and lag_24 (rentals from this exact hour yesterday). This transforms the time-series problem into a tabular format the Random Forest can understand.
Rolling Momentum: I created a rolling_mean_24h feature to capture the short-term momentum of the day. To prevent data leakage (giving the model the answer before it happens), this rolling average was calculated after shifting the data by one hour.

## Modeling:
a) Choose any model you find appropriate and justify your choice
I chose a Random Forest Regressor for this task. Bike rental demand is driven by complex, non-linear interactions (e.g., the combination of "8:00 AM" AND "Weekday" causes a massive spike, while "8:00 AM" AND "Weekend" does not). Random Forests naturally capture these conditional interactions without requiring heavy feature scaling. Additionally, setting n_estimators=100 created a robust ensemble that is highly resistant to overfitting the noisy daily variations.

b) Split data in a time-aware manner no random splits
In standard machine learning, data is typically shuffled randomly before splitting. However, in time series forecasting, random shuffling causes "data leakage" (allowing the model to peek into the future to predict the past). To mimic a real-world production environment, I performed a strict chronological split. The first 80% of the timeline (approx. 13,800 hours) was used as the training set, and the final 20% (approx. 3,400 hours) was held out as unseen future data for testing.

## Evaluation
a) Report MAE and RMSE on the held-out set
b) Compare your model's predictions against a simple baseline
I evaluated the Random Forest against a Persistence Baseline, which operates on the assumption that the next hour's demand will exactly match the current hour's demand. In a time series forecasting problem, this is generally a tough baseline to beat in the short term. The baseline achieved an MAE of 85.12 bikes and an RMSE of 129.54 bikes. The Random Forest Regressor drastically outperformed the baseline, achieving an MAE of 32.02 bikes and an RMSE of 53.80 bikes.
The model improved upon the baseline by an average of over 53 bikes per hour. The persistence baseline fails because it cannot anticipate sudden behavioral shifts (like the transition from a quiet 7:00 AM to a chaotic 8:00 AM rush hour). Because the Random Forest was fed engineered features like workingday, hr, weather conditions, and lag features, it successfully anticipated these surges. For Capital Bikeshare, an improvement of this magnitude translates directly into more efficient fleet rebalancing, fewer empty docks during peak hours, and ultimately, higher revenue and customer satisfaction.
