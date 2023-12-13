# Lights Out: Forecasting Residential Impact of Power Outages
In this project, we are tackling a regression problem. Our goal is to predict the percentage of residential customers (RES.CUST.PCT) affected by a major power outage. We chose this as our response variable because it provides a quantifiable measure of the impact of power outages on residential customers.

(CHANGE LINE AS SOON AS ANDREW PUSHES HIS CODE)
The features we are using to train our model include U.S._STATE and OUTAGE.DURATION. These features were chosen because they are known at the “time of prediction” and are expected to have a significant influence on the percentage of residential customers affected by an outage.

## Model Evaluation
To evaluate our model, we are using the Root Mean Squared Error (RMSE). This metric was chosen because it is suitable for regression problems and it penalizes large errors more due to the squaring of the residuals. This makes it a good choice for our problem where we want to minimize the difference between the actual and predicted percentages of residential customers affected by an outage.

## Feature Selection
Our target variable is RES.CUST.PCT, which represents the percentage of residential customers affected by a major power outage. To predict this, we’ve selected two features: U.S._STATE and OUTAGE.DURATION.

- U.S._STATE: This is a categorical variable representing the U.S. state where the outage occurred. The rationale behind including this feature is that the impact of an outage can vary by location due to factors such as infrastructure, population density, and local policies. By including the state as a feature, our model can learn these regional differences.
- OUTAGE.DURATION: This is a numerical variable representing the duration of the power outage. It’s reasonable to assume that longer outages will affect more customers, making this a potentially powerful feature for our prediction problem.
- ADD ANDREW"S FEATURES LATER

## Model Description and Feature Selection

Our baseline model is a Linear Regression model that predicts the percentage of residential customers (RES.CUST.PCT) affected by a major power outage. 

Initially, we considered three features: `U.S._STATE`, `OUTAGE.DURATION`, and `POPULATION`. After a thorough analysis and understanding of the data, we decided to focus on two features: `U.S._STATE` and `OUTAGE.DURATION`.

- **U.S._STATE (Nominal):** This categorical feature represents the U.S. state where the outage occurred. It’s encoded using one-hot encoding, which creates a binary variable for each state. This categorical feature represents the U.S. state where the outage occurred. The rationale behind including this feature is that the impact of an outage can vary by location due to factors such as infrastructure, population density, and local policies. By including the state as a feature, our model can learn these regional differences.

- **OUTAGE.DURATION (Quantitative):** This numerical feature represents the duration of the power outage in minutes. It’s scaled using standard scaling, which standardizes the feature by subtracting the mean and scaling to unit variance. This numerical feature represents the duration of the power outage in minutes. It’s reasonable to assume that longer outages will affect more customers, making this a potentially powerful feature for our prediction problem.

The `POPULATION` feature was excluded after considering several factors:

- **Relevance:** While the population of a state could potentially influence the percentage of residential customers affected by a power outage, our analysis shown that `U.S._STATE` and `OUTAGE.DURATION` were more strongly related to the target variable.

- **Redundancy:** The `POPULATION` feature was highly correlated with the `U.S._STATE` feature, providing little additional information.

By focusing on `U.S._STATE` and `OUTAGE.DURATION`, we were able to build a model that captures the key factors affecting the percentage of residential customers impacted by a power outage, while also being relatively simple and interpretable. Thus, our model includes one nominal feature and one quantitative feature. There are no ordinal features in our model.

<iframe src="Assets/3d_plot.html" width=800 height=600 frameBorder=0></iframe>

The 3D scatter plot visualizes the relationship between `OUTAGE.DURATION`, `POPULATION`, and the predicted percentage of residential customers (`PREDICTED`) affected by a major power outage. The data points are color-coded by `U.S._STATE`, meaning each point represents a different state in the USA. This plot provides a visual way to understand the model’s predictions and the relationships between these three variables. The color-coding by state in the 3D plot allows us to see if certain states have higher or lower percentages of residential customers affected by outages. The `OUTAGE.DURATION` trend along the x-axis suggests that the duration of the outage has an impact on the predicted percentage of residential customers affected. 


## Data Preprocessing

Before training the model, we preprocess the data:

- **Handling Missing Values:** We fill missing values in numerical columns with the mean of the respective column and in categorical columns with the mode (most frequent value) of the respective column.

- **Converting Date/Time Columns:** We convert the OUTAGE.START and OUTAGE.RESTORATION columns to datetime format and create a new feature for outage duration in minutes.

- **Encoding Categorical Variables:** We encode the categorical variables using scikit-learn’s LabelEncoder.

```python
from sklearn.preprocessing import LabelEncoder

processed_outage = outage_cleaned.copy()

# Fill numerical columns with the mean
num_cols = processed_outage.select_dtypes(include=[np.number]).columns
processed_outage[num_cols] = processed_outage[num_cols].fillna(processed_outage[num_cols].mean())

# Fill categorical columns with the mode
cat_cols = processed_outage.select_dtypes(include=['object']).columns
processed_outage[cat_cols] = processed_outage[cat_cols].fillna(processed_outage[cat_cols].mode().iloc[0])

# 2. Convert Date/Time Columns
processed_outage['OUTAGE.START'] = pd.to_datetime(processed_outage['OUTAGE.START'])
processed_outage['OUTAGE.RESTORATION'] = pd.to_datetime(processed_outage['OUTAGE.RESTORATION'])

# Create a new feature for outage duration in minutes
processed_outage['OUTAGE.DURATION'] = (processed_outage['OUTAGE.RESTORATION'] - processed_outage['OUTAGE.START']).dt.total_seconds() / 60

# 3. Encode Categorical Variables
le = LabelEncoder()
for col in cat_cols:
    processed_outage[col] = le.fit_transform(processed_outage[col])
```

The head of this processed table looks like so:

|   OBS |   YEAR |   MONTH |   U.S._STATE |   POSTAL.CODE |   NERC.REGION |   CLIMATE.REGION |   ANOMALY.LEVEL |   CLIMATE.CATEGORY |   CAUSE.CATEGORY |   CAUSE.CATEGORY.DETAIL |   HURRICANE.NAMES |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   RES.CUSTOMERS |   COM.CUSTOMERS |   IND.CUSTOMERS |   TOTAL.CUSTOMERS |   RES.CUST.PCT |   COM.CUST.PCT |   IND.CUST.PCT |   PC.REALGSP.STATE |   PC.REALGSP.USA |   PC.REALGSP.REL |   PC.REALGSP.CHANGE |   UTIL.REALGSP |   TOTAL.REALGSP |   UTIL.CONTRI |   PI.UTIL.OFUSA |   POPULATION |   POPPCT_URBAN |   POPPCT_UC |   POPDEN_URBAN |   POPDEN_UC |   POPDEN_RURAL |   AREAPCT_URBAN |   AREAPCT_UC |   PCT_LAND |   PCT_WATER_TOT |   PCT_WATER_INLAND | OUTAGE.START        | OUTAGE.RESTORATION   |
|------:|-------:|--------:|-------------:|--------------:|--------------:|-----------------:|----------------:|-------------------:|-----------------:|------------------------:|------------------:|------------------:|-----------------:|---------------------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|:--------------------|:---------------------|
|     0 |     11 |       6 |           23 |            23 |             6 |                1 |              13 |                  1 |                5 |                      44 |                20 |               570 |                0 |                  201 |         328 |         225 |         237 |           270 |         342 |         337 |         487 |           346 |          444 |          279 |          729 |             177 |             148 |             176 |               175 |            343 |             66 |            225 |                275 |                7 |              286 |                  63 |            173 |             191 |           157 |              20 |          158 |             24 |          31 |             33 |          35 |             16 |              17 |           17 |         16 |              33 |                 43 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |
|     1 |     14 |       4 |           23 |            23 |             6 |                1 |              15 |                  1 |                2 |                      44 |                20 |                 1 |                0 |                    0 |         363 |         269 |         210 |           270 |         211 |         272 |         445 |           275 |           99 |          395 |          855 |             180 |             157 |             161 |               179 |            331 |             76 |            209 |                321 |               13 |              306 |                  66 |            196 |             208 |           171 |              20 |          163 |             24 |          31 |             33 |          35 |             16 |              17 |           17 |         16 |              33 |                 43 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |
|     2 |     10 |       9 |           23 |            23 |             6 |                1 |               1 |                  0 |                5 |                      20 |                20 |               562 |                0 |                  201 |         268 |         146 |         171 |           173 |         199 |         269 |         462 |           269 |           51 |          412 |          885 |             176 |             149 |             165 |               174 |            342 |             69 |            217 |                266 |                6 |              269 |                  74 |            166 |             183 |           135 |              19 |          157 |             24 |          31 |             33 |          35 |             16 |              17 |           17 |         16 |              33 |                 43 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |
|     3 |     12 |       5 |           23 |            23 |             6 |                1 |              15 |                  1 |                5 |                      36 |                20 |               522 |                0 |                  194 |         338 |         230 |         228 |           264 |         260 |         303 |         467 |           304 |          194 |          354 |          826 |             178 |             153 |             182 |               176 |            341 |             68 |            229 |                281 |                9 |              278 |                  53 |            203 |             195 |           235 |              20 |          159 |             24 |          31 |             33 |          35 |             16 |              17 |           17 |         16 |              33 |                 43 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |
|     4 |     15 |       6 |           23 |            23 |             6 |                1 |              28 |                  2 |                5 |                      44 |                20 |               452 |              114 |                  460 |         427 |         303 |         312 |           352 |         288 |         343 |         432 |           314 |          331 |          524 |          625 |             181 |             158 |             160 |               181 |            328 |             78 |            204 |                333 |               15 |              310 |                  64 |            177 |             209 |           122 |              20 |          165 |             24 |          31 |             33 |          35 |             16 |              17 |           17 |         16 |              33 |                 43 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |

## Model Training and Evaluation

We train the model using a training set and evaluate its performance on a test set. The performance is measured using the root mean squared error (RMSE), a common metric for regression problems that measures the average magnitude of the prediction error.

```python
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

# Select features and target
features = processed_outage[['U.S._STATE', 'OUTAGE.DURATION']]
target = processed_outage['RES.CUST.PCT']

# Split the data into training and test sets
X_train, X_test, y_train, y_test = train_test_split(features, target, test_size=0.3, random_state=42)

# Define preprocessing for numerical columns (scale them)
num_processor = Pipeline([
    ('scaler', StandardScaler())
])

# Define preprocessing for categorical columns (encode them)
cat_processor = Pipeline([
    ('encoder', OneHotEncoder(handle_unknown='ignore'))
])

# Combine preprocessing steps
preprocessor = ColumnTransformer(
    transformers=[
        ('num', num_processor, ['OUTAGE.DURATION']),
        ('cat', cat_processor, ['U.S._STATE'])
    ])

# Create the pipeline
pipeline = Pipeline(steps=[
    ('preprocessor', preprocessor),
    ('regressor', LinearRegression())
])

# Train the model
pipeline.fit(X_train, y_train)
```

# Model Performance and Evaluation

## Performance Metrics

The performance of our model is evaluated using the Root Mean Squared Error (RMSE), a common metric for regression problems that measures the average magnitude of the prediction error.

## Training Performance

Our model achieved an RMSE of 23.27 on the training set. This means that, on average, the model’s predictions on the training data were about 23.27 units away from the true values.

## Test Performance

On the test set, the model achieved an RMSE of 21.38. This lower RMSE indicates that the model’s predictions were, on average, about 21.38 units away from the true values on unseen data, which is slightly better than its performance on the training set.

## Visual Evaluation
<iframe src="Assets/regression_line.html" width=800 height=600 frameBorder=0></iframe>

The scatter plot provided shows the model’s predicted values versus the actual values. Each point represents a power outage event, with its position along the x-axis showing the actual percentage of residential customers affected, and its position along the y-axis showing the model’s prediction.

The red line represents perfect predictions, where the predicted value is equal to the actual value. Points that are closer to this line represent more accurate predictions.

From the plot, we can see that the model’s predictions are generally close to the actual values, indicating that the model is performing well. 

## Model Assessment

Given the RMSE values and the scatter plot, we can conclude that our model is performing reasonably well. It’s able to predict the percentage of residential customers affected by a major power outage with a reasonable level of accuracy. In our case, the current level of accuracy is acceptable for the task because the RMSE values we obtained (23.27 on the training set and 21.38 on the test set) are small enough to be practically insignificant. A prediction error of this magnitude will not significantly affect the decisions or actions that are based on the model’s predictions. Thus, our model could be considered “good”.
