# Lights Out: Forecasting Residential Impact of Power Outages
by Luke Lin & Andrew Yin

## Framing the Problem
In this project, we are tackling a regression problem. Our goal is to predict the percentage of residential customers (RES.CUST.PCT) affected by a major power outage. We chose this as our response variable because it provides a quantifiable measure of the impact of power outages on residential customers.

The features we are using to train our model include `U.S._STATE`, `OUTAGE.DURATION`, `DEMAND.LOSS.MW`, `POPULATION`, and `CUSTOMERS.AFFECTED`. These features were chosen because they are known at the time of prediction and based on our assumptions, they are expected to have a significant influence on the percentage of residential customers affected by an outage.

### Model Evaluation
To evaluate our model, we are using the Root Mean Squared Error (RMSE). This metric was chosen because it is suitable for regression problems and it penalizes large errors more due to the squaring of the residuals. This makes it a good choice for our problem where we want to minimize the difference between the actual and predicted percentages of residential customers affected by an outage.

### Feature Selection
Our target variable is RES.CUST.PCT, which represents the percentage of residential customers affected by a major power outage. To predict this, we’ve selected several features: 

- `U.S._STATE`: This is a categorical variable representing the U.S. state where the outage occurred. The rationale behind including this feature is that the impact of an outage can vary by location due to factors such as infrastructure, population density, and local policies. By including the state as a feature, our model can learn these regional differences.
- `OUTAGE.DURATION`: This is a numerical variable representing the duration of the power outage. It’s reasonable to assume that longer outages will affect more customers, making this a potentially powerful feature for our prediction problem.
- `DEMAND.LOSS.MW`: The amount of peak demand lost during an outage.
- `POPULATION`: The population in the area of the outage.
- `CUSTOMERS.AFFECTED`: Number of customers affected by the outage.

## Baseline Model

### Cleaning the Data
In this section, we’ve performed a series of data cleaning steps on the outage DataFrame, which was read from an Excel file named “outage.xlsx”. This process was a direct replication of the cleaning procedure implemented in Project 3, ensuring consistency and reliability in our data preparation phase.

Initially, we removed informational rows and columns that contained only null values from the DataFrame. We then set the column names based on the first row of the DataFrame for better readability and understanding of the data.

Next, we dropped rows related to units and variables that were not necessary for our analysis. We also dropped the “variables” column as it was not needed.

One of the crucial steps in this process was the creation of new datetime columns, OUTAGE.START and OUTAGE.RESTORATION. These were formed by combining the respective date and time columns, providing us with precise timestamps of when the outage started and ended. After creating these new columns, the original date and time columns were dropped to avoid redundancy.

Lastly, we replaced the “NA” entries with NaN for missing values.

After these cleaning steps, we were left with a cleaned DataFrame, outage_cleaned, ready for further analysis or modeling.

Our exploratory data analysis on this dataset can be found [Empowering Resilience: Unraveling the Tapestry of Power Outages Over a Decade](https://lukelin15.github.io/Power-Outage-Analysis/).

### Model Description and Feature Selection

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


### Data Preprocessing

Before training the model, we preprocess the data:

- **Handling Missing Values:** We fill missing values in numerical columns with the mean of the respective column and in categorical columns with the mode (most frequent value) of the respective column.

- **Converting Date/Time Columns:** We convert the OUTAGE.START and OUTAGE.RESTORATION columns to datetime format and create a new feature for outage duration in minutes.

```python
processed_outage = outage_cleaned.copy()

# Convert Date/Time Columns
processed_outage['OUTAGE.START'] = pd.to_datetime(processed_outage['OUTAGE.START'])
processed_outage['OUTAGE.RESTORATION'] = pd.to_datetime(processed_outage['OUTAGE.RESTORATION'])

# Create a new feature for outage duration in minutes
processed_outage['OUTAGE.DURATION'] = (processed_outage['OUTAGE.RESTORATION'] - processed_outage['OUTAGE.START']).dt.total_seconds() / 60

# Fill numerical columns with the mean
num_cols = ['OUTAGE.DURATION', 'DEMAND.LOSS.MW', 'POPULATION', 'CUSTOMERS.AFFECTED']
for col in num_cols:
    processed_outage[col] = processed_outage[col].fillna(processed_outage[col].mean())

# Fill categorical columns with the mode
cat_cols = ['U.S._STATE']
processed_outage[cat_cols] = processed_outage[cat_cols].fillna(processed_outage[cat_cols].mode().iloc[0])

# Convert RES.CUST.PCT to percentage 
processed_outage["RES.CUST.PCT"] = processed_outage["RES.CUST.PCT"] / 100 

processed_outage = processed_outage[['U.S._STATE', 'OUTAG
```

The head of this processed table looks like so:

| U.S._STATE   |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   POPULATION |   CUSTOMERS.AFFECTED |   RES.CUST.PCT |
|:-------------|------------------:|-----------------:|-------------:|---------------------:|---------------:|
| Minnesota    |              3060 |          536.287 |      5348119 |                70000 |       0.889448 |
| Minnesota    |                 1 |          536.287 |      5457125 |               143456 |       0.888335 |
| Minnesota    |              3000 |          536.287 |      5310903 |                70000 |       0.889206 |
| Minnesota    |              2550 |          536.287 |      5380443 |                68200 |       0.888954 |
| Minnesota    |              1740 |          250     |      5489594 |               250000 |       0.888216 |

### Model Training and Evaluation

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

Our model achieved an RMSE of 0.0041986 on the training set. This means that, on average, the model’s predictions on the training data were about 0.0041986 units away from the true values.

## Test Performance

On the test set, the model achieved an RMSE of 0.00325. This lower RMSE indicates that the model’s predictions were, on average, about 0.00325 units away from the true values on unseen data, which is slightly better than its performance on the training set.

## Visual Evaluation
<iframe src="Assets/regression_line.html" width=800 height=600 frameBorder=0></iframe>

The scatter plot provided shows the model’s predicted values versus the actual values. Each point represents a power outage event, with its position along the x-axis showing the actual percentage of residential customers affected, and its position along the y-axis showing the model’s prediction.

The red line represents perfect predictions, where the predicted value is equal to the actual value. Points that are closer to this line represent more accurate predictions.

From the plot, we can see that the model’s predictions are generally close to the actual values, indicating that the model is performing well. 

### Model Assessment

Given the RMSE values and the scatter plot, we can conclude that our model is performing reasonably well. It’s able to predict the percentage of residential customers affected by a major power outage with a reasonable level of accuracy. In our case, the current level of accuracy is acceptable for the task because the RMSE values we obtained (0.004198 on the training set and 0.003259 on the test set) are small enough to be practically insignificant. A prediction error of this magnitude will not significantly affect the decisions or actions that are based on the model’s predictions. Thus, our model could be considered “good”.


# Final Model

## New Features

In the development of the final model, three new features were added to the existing dataset: 'DEMAND.LOSS.MW', 'POPULATION', and 'CUSTOMERS.AFFECTED'. These features were carefully selected based on their relevance and potential impact on the prediction task, which is forecasting the 'RES.CUST.PCT' (percentage of residential customers affected by power outages). Below is an explanation of why each of these features is believed to improve the model’s performance:

1. **'DEMAND.LOSS.MW' (Demand Loss in Megawatt)**
   - **Rationale**: This feature represents the amount of power demand lost during an outage, measured in Megawatts. It is a direct indicator of the severity of an outage. Higher demand loss likely correlates with more significant impact on residential customers, making it a valuable predictor for the percentage of affected customers.
   - **Impact on Model**: By quantifying the outage's impact in terms of lost power, this feature provides the model with a measurable scale of outage severity, which is expected to have a direct relationship with 'RES.CUST.PCT'.

2. **'POPULATION'**
   - **Rationale**: The total population in the affected area can be a crucial factor in understanding the extent of an outage's impact. Areas with higher populations might have more robust infrastructure to handle outages, or conversely, might see a higher percentage of affected customers due to the sheer number of people involved.
   - **Impact on Model**: Including population allows the model to account for the scale of potential impact, offering a context for the outage data. The use of `QuantileTransformer` on this feature helps in normalizing its distribution and mitigating the influence of outliers, thereby making the model more robust.

3. **'CUSTOMERS.AFFECTED'**
   - **Rationale**: This feature directly measures the number of customers affected by each outage event. It serves as a straightforward indicator of the outage's impact and provides a clear link to the target variable 'RES.CUST.PCT'.
   - **Impact on Model**: The inclusion of 'CUSTOMERS.AFFECTED' gives the model explicit information about the extent of each outage's impact, potentially improving its accuracy in predicting the percentage of affected residential customers.

The addition of these features is based on the understanding that the severity of a power outage (represented by 'DEMAND.LOSS.MW'), the demographic context ('POPULATION'), and the direct impact ('CUSTOMERS.AFFECTED') are all critical factors in predicting the proportion of affected residential customers. These features were expected to provide the model with comprehensive and relevant information, leading to more accurate and reliable predictions.

## Overview of the Modeling Algorithm
The final model is built using a RandomForestRegressor. RandomForestRegressor operates by constructing a multitude of decision trees at training time and outputting the average prediction of the individual trees. This approach helps in reducing overfitting, improving the model's generalizability, and handling non-linear relationships between features and the target variable effectively.

## Features and Preprocessing
In addition to the original features ('U.S._STATE' and 'OUTAGE.DURATION'), the model integrates three new features: 'DEMAND.LOSS.MW', 'POPULATION', and 'CUSTOMERS.AFFECTED'. The preprocessing pipeline includes:
- **StandardScaler**: Applied to 'OUTAGE.DURATION' and 'DEMAND.LOSS.MW' for normalizing these features, which helps in improving the model's performance.
- **QuantileTransformer**: Used on 'POPULATION' and 'CUSTOMERS.AFFECTED' to transform these features to follow a normal distribution, thereby mitigating the effects of outliers and skewness in the data.
- **OneHotEncoder**: Employed for the categorical variable 'U.S._STATE', enabling the model to better understand and use this non-numeric information.

## Hyperparameter Tuning
The hyperparameters of the RandomForestRegressor were tuned using GridSearchCV, an exhaustive search over specified parameter values. The following hyperparameters were considered:
- **n_estimators**: Number of trees in the forest. Tested values were [100, 200].
- **max_depth**: Maximum depth of the trees. Tested values were [10, 20].
- **min_samples_split**: Minimum number of samples required to split an internal node. Tested values were [2, 5].

The best performing hyperparameters were determined based on the lowest Root Mean Squared Error (RMSE) achieved during the cross-validation process in GridSearchCV. These were:
- **n_estimators**: 200
- **max_depth**: 20
- **min_samples_split**: 2

## Model
```python
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder, QuantileTransformer
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error

# Select features and target
features = processed_outage[['U.S._STATE', 'OUTAGE.DURATION', 'DEMAND.LOSS.MW', 'POPULATION', 'CUSTOMERS.AFFECTED']]
target = processed_outage['RES.CUST.PCT']

# Split the data
X_train, X_test, y_train, y_test = train_test_split(features, target, test_size=0.3, random_state=42)

# Preprocessing for numerical columns
num_processor = Pipeline([
    ('scaler', StandardScaler())
])

# Preprocessing for numerical columns using quantiles
qnt_processor = Pipeline([
    ('quantile', QuantileTransformer(n_quantiles=400, output_distribution='normal'))
])

# Preprocessing for categorical columns
cat_processor = Pipeline([
    ('encoder', OneHotEncoder(handle_unknown='ignore'))
])

# Combine preprocessing
preprocessor = ColumnTransformer(
    transformers=[
        ('num', num_processor, ['OUTAGE.DURATION', 'DEMAND.LOSS.MW']),
        ('qnt', qnt_processor, ['POPULATION', 'CUSTOMERS.AFFECTED']),
        ('cat', cat_processor, ['U.S._STATE'])
    ])

# Define the model
model = RandomForestRegressor()

# Create the pipeline
pipeline = Pipeline(steps=[
    ('preprocessor', preprocessor),
    ('model', model)
])

# Define the grid search
param_grid = {
    'model__n_estimators': [100, 200],
    'model__max_depth': [10, 20],
    'model__min_samples_split': [2, 5]
}
grid_search = GridSearchCV(pipeline, param_grid, cv=5)

# Fit the model
grid_search.fit(X_train, y_train)

# Best model
best_model = grid_search.best_estimator_

# Evaluate the model
y_pred = best_model.predict(X_test)
```

## Performance Improvement Over Baseline Model
The final model demonstrated a significant improvement in performance over the baseline model, as evidenced by a lower RMSE of 8.6. This improvement can be attributed to the inclusion of additional relevant features and the application of appropriate preprocessing techniques, which enhanced the model's ability to capture and learn from the complexities in the data. Moreover, the optimized hyperparameters via GridSearchCV further refined the model's predictive capabilities, ensuring a better fit to the data while maintaining generalizability. Below is a visual of the predictions vs actual for newer model. The smaller spread and decrease in outliers demonstrates a stronger model

<iframe src="Assets/new_regression.html" width=800 height=600 frameBorder=0></iframe>

The use of RandomForestRegressor, with its inherent mechanisms for handling overfitting and non-linearity, combined with meticulous feature engineering and hyperparameter optimization, led to a robust and effective final model for predicting the percentage of residential customers affected by power outages.

# Fairness Analysis

## Group Definition
- **Group X (Northern Half)**: This group consists of data from the 'Northeast', 'Northwest', 'East North Central', and 'West North Central' climate regions. These regions are categorized as the Northern Half of the United States.
- **Group Y (Everything Else)**: This group comprises data from all other climate regions, representing the rest of the United States.

## Evaluation Metric
The evaluation metric used is the Root Mean Squared Error (RMSE), a standard measure for assessing the accuracy of a regression model. It quantifies the difference between the predicted values and the actual values.

## Hypotheses
- **Null Hypothesis (H0)**: The model is fair across the different climate regions. The RMSE for Group X and Group Y are roughly the same, implying any observed differences are due to random chance.
- **Alternative Hypothesis (H1)**: The model is unfair, showing a significant difference in RMSE between Group X (Northern Half) and Group Y (Everything Else).

## Test Statistic and Significance Level
The test statistic is the absolute difference in RMSE between the two groups. The permutation test involves shuffling the 'CLIMATE.REGION' labels and recalculating this difference. The significance level we choose is 0.05 as it is a common metric of significance.

## P-Value and Conclusion
After running the permutation test for 1000 iterations, the p-value is calculated as the proportion of permutations where the observed test statistic is at least as extreme as the one calculated from the original dataset.

**P-Value**: 0.054

**Conclusion**: With a p-value of 0.054, which is slightly above the commonly used significance level of 0.05, we fail to reject the null hypothesis. This suggests that there is no significant evidence of unfairness in the model's performance between the Northern Half and the rest of the United States based on our statistical test. However, the p-value is close to the threshold, indicating that the possibility of a difference in model performance between these groups cannot be entirely dismissed. Further investigation or additional data might be required for a more conclusive understanding.

