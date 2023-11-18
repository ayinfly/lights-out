# Analysis of Power Outages

by Luke Lin (lul018@ucsd.edu) & Andrew (anyin@ucsd.edu)

---

## Introduction 

Welcome to our exploration of a comprehensive dataset detailing power outages across various U.S. states. Power outages are critical events that impact individuals, communities, and businesses, influencing everything from daily routines to economic activities. Understanding the dynamics of power outages is crucial for devising strategies to improve outage management and enhance the resilience of power infrastructure.

The dataset at the heart of our analysis provides a wealth of information about each power outage, encompassing details such as the cause of the outage, its duration, the number of customers affected, and additional factors that contribute to the complexity of outage events. By delving into this dataset, we aim to answer a pivotal question: “How has outage handling improved over the last decade?”

Why should you, as a reader, care about this dataset and our central question? Power outages are not merely inconveniences; they have far-reaching implications for individuals, businesses, and society at large. An examination of how outage handling practices have evolved over time can offer insights into the efficacy of measures implemented to mitigate outage impacts. This analysis is particularly relevant in a world increasingly dependent on a consistent and reliable power supply for various aspects of daily life.

## Dataset Overview

Our dataset encompasses over 1000 rows, each representing a distinct power outage event. The columns relevant to our central question include:

### U.S._STATE
- **Description:** The state where the power outage occurred.
- **Purpose:** To identify the geographic location of each power outage event.

### OUTAGE.START & OUTAGE.RESTORATION
- **Description:** The timestamp indicating when the power outage started and restored.
- **Calculation:** Derived by combining 'OUTAGE.START.DATE' and 'OUTAGE.START.TIME' into a single datetime column.
- **Calculation:** Derived by combining 'OUTAGE.RESTORATION.DATE' and 'OUTAGE.RESTORATION.TIME' into a single datetime column.
- **Purpose:** To determine when the power was restored after an outage, providing insights into outage recovery times.

### CUSTOMERS.AFFECTED
- **Description:** The number of customers affected by the power outage.
- **Purpose:** Quantifies the impact of the outage in terms of the affected customer base.

### OUTAGE.DURATION
- **Description:** The duration of the power outage in minutes.
- **Purpose:** Measures the length of time that the power outage persisted, providing crucial information for outage management and planning.

### DEMAND.LOSS.MW
- **Description:** The amount of demand loss in megawatts during the power outage.
- **Purpose:** Indicates the reduction in electrical demand during the outage, helping to assess the magnitude of the impact.

### RES.PRICE
- **Description:** The residential electricity price during the outage.
- **Purpose:** Provides information on the residential electricity pricing during the outage event.

### NERC.REGION
- **Description:** The region assigned by the North American Electric Reliability Corporation (NERC) where the outage occurred.
- **Purpose:** Helps categorize and analyze outages based on regional factors.

### CLIMATE.REGION
- **Description:** The climate region associated with the outage location.
- **Purpose:** Considers climate-related factors that might influence power outages.

### CAUSE.CATEGORY
- **Description:** The category specifying the cause of the power outage.
- **Purpose:** Classifies outages based on the reasons for their occurrence, aiding in root cause analysis.

### TOTAL.SALES
- **Description:** Total electricity sales during the outage.
- **Purpose:** Provides information on the overall electricity sales during the outage period.

Here’s the head of the DataFrame:

| U.S._STATE   | OUTAGE.START        | OUTAGE.RESTORATION   |   CUSTOMERS.AFFECTED |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   RES.PRICE | NERC.REGION   | CLIMATE.REGION     | CAUSE.CATEGORY     |   TOTAL.SALES |
|:-------------|:--------------------|:---------------------|---------------------:|------------------:|-----------------:|------------:|:--------------|:-------------------|:-------------------|--------------:|
| Minnesota    | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |                70000 |              3060 |              nan |       11.6  | MRO           | East North Central | severe weather     |       6562520 |
| Minnesota    | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |                  nan |                 1 |              nan |       12.12 | MRO           | East North Central | intentional attack |       5284231 |
| Minnesota    | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |                70000 |              3000 |              nan |       10.87 | MRO           | East North Central | severe weather     |       5222116 |
| Minnesota    | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |                68200 |              2550 |              nan |       11.79 | MRO           | East North Central | severe weather     |       5787064 |
| Minnesota    | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |               250000 |              1740 |              250 |       13.07 | MRO           | East North Central | severe weather     |       5970339 |

In the next sections, we will illustrate the process through which we obtained this cleaned dataframe. Additionally, we will employ compelling visualizations to delve deeper into our primary research question.

## Cleaning and EDA

We started by cleaning the dataset, handling missing values, and converting data types where necessary. We then performed an exploratory data analysis to understand the distribution of power outages across different states, causes, and time periods.

### Step 1: Read the Excel file into a pandas DataFrame

```python
outage = pd.read_excel("outage.xlsx", sheet_name="Masterdata")
```

The dataset for our power outage analysis is stored in an Excel file. The conventional read_csv method, which we learned in class, was not applicable in this case. Therefore, we used the read_excel method and specified the sheet name to ensure that the output is a pandas dataframe, rather than a generic Excel object. Now, we have a pandas dataframe ready for further cleaning and analysis.

### Step 2: Drop Informational Rows

```python
outage_cleaned = outage.drop(range(4)).dropna(axis=1, how='all')
```

In this step, we eliminated the initial rows from the Excel file. These rows contained the title of the Excel file, which was not relevant to our analysis and disrupted the desired column and row structure.

### Step 3: Set Column Names Based on the First Row and  Remove Rows and Columns Related to Units and Variables

```python
outage_cleaned.columns = outage_cleaned.iloc[0]
outage_cleaned = outage_cleaned.drop([4, 5])
outage_cleaned = outage_cleaned.drop(columns="variables")
```

With the removal of the initial rows, the first row of the dataframe now contains our desired column names. We assigned these as our column names. After correctly setting the column names, we removed the first row from the dataframe. We also eliminated the subsequent row, which merely specified the unit for each column.

### Step 4: Combining START and RESTORATION Times

```python
outage_cleaned['OUTAGE.START'] = pd.to_datetime(outage_cleaned['OUTAGE.START.DATE']) + pd.to_timedelta(outage_cleaned['OUTAGE.START.TIME'].astype(str))
outage_cleaned['OUTAGE.RESTORATION'] = pd.to_datetime(outage_cleaned['OUTAGE.RESTORATION.DATE']) + pd.to_timedelta(outage_cleaned['OUTAGE.RESTORATION.TIME'].astype(str))
outage_cleaned = outage_cleaned.drop(['OUTAGE.START.DATE', 'OUTAGE.START.TIME', 'OUTAGE.RESTORATION.DATE', 'OUTAGE.RESTORATION.TIME'], axis=1)
```

To avoid redundancy and retain all necessary information, we combined the START and RESTORATION times into single columns, subsequently dropping the original columns.

### Step 5: Essential Columns Analysis

```python
essential_col = [
    "U.S._STATE", 
    'OUTAGE.START',
    "OUTAGE.RESTORATION", 
    "CUSTOMERS.AFFECTED", 
    "OUTAGE.DURATION", 
    "DEMAND.LOSS.MW", 
    "RES.PRICE", 
    "NERC.REGION", 
    "CLIMATE.REGION",
    "CAUSE.CATEGORY",
    "TOTAL.SALES"
]

outage_cleaned = outage_cleaned[essential_col]
```

The above code retains only the columns essential for our analysis, ensuring a focused and efficient examination of the data.

## Univariate Analysis
We first looked at the distribution of the OUTAGE.DURATION column. The plot below shows that most power outages last less than 10k minutes (or about 7 days), but there are some outages that last much longer.
<iframe src="Assets/Distribution_of_OUTAGE.html" width=800 height=600 frameBorder=0></iframe>

## Bivariate Analysis
We further investigated the relationship between the duration of power outages ('OUTAGE.DURATION') and the number of customers affected ('CUSTOMERS.AFFECTED'). The scatter plot below illustrates a general trend where shorter outages tend to have fewer affected customers, while longer outages show a wider range of customer impacts. This suggests that the duration of an outage might be a factor in determining the scale of its impact on customers.
<iframe src="Assets/scatter.html" width=800 height=600 frameBorder=0></iframe>

## Interesting Aggregates
After seeing large swings in power outage durations, we also examined the average number of customers affected for each state. The table below shows that the average outage duration varies widely from state to state. The table below also includes the latitude and longtitude of each state to display on a heap map using Folium.


| U.S._STATE           |   CUSTOMERS.AFFECTED |   Latitude |   Longitude |
|:---------------------|---------------------:|-----------:|------------:|
| Alabama              |             94328.8  |    33.2589 |    -86.8295 |
| Alaska               |             14273    |    64.446  |   -149.681  |
| Arizona              |             64402.7  |    34.3953 |   -111.763  |
| Arkansas             |             47673.8  |    35.2049 |    -92.4479 |
| California           |            201366    |    36.7015 |   -118.756  |
| Colorado             |             41060.6  |    38.7252 |   -105.608  |
| Connecticut          |             60339.2  |    41.65   |    -72.7342 |
| Delaware             |              3475    |    38.692  |    -75.4013 |
| District of Columbia |            194709    |    38.8938 |    -76.988  |
| Florida              |            289369    |    27.7568 |    -81.464  |
| Georgia              |            120680    |    32.3294 |    -83.1137 |
| Hawaii               |            147237    |    19.5938 |   -155.428  |
| Idaho                |              5833.33 |    43.6448 |   -114.015  |
| Illinois             |            207027    |    40.0797 |    -89.4337 |
| Indiana              |             69551.4  |    40.327  |    -86.1747 |
| Iowa                 |             94000    |    41.9217 |    -93.3123 |
| Kansas               |            108000    |    38.2731 |    -98.5822 |
| Kentucky             |            130531    |    37.5726 |    -85.1551 |
| Louisiana            |            151003    |    30.8704 |    -92.0071 |
| Maine                |             54839.4  |    45.7091 |    -68.859  |
| Maryland             |            120535    |    39.5162 |    -76.9382 |
| Massachusetts        |             77983.4  |    42.3789 |    -72.0324 |
| Michigan             |            152878    |    43.6212 |    -84.6824 |
| Minnesota            |            124007    |    45.9897 |    -94.6113 |
| Mississippi          |              5000    |    32.9715 |    -89.7348 |
| Missouri             |             50611.1  |    38.7605 |    -92.5618 |
| Nebraska             |             87070.7  |    41.737  |    -99.5874 |
| Nevada               |             22220    |    39.5159 |   -116.854  |
| New Hampshire        |             13869.8  |    43.4849 |    -71.6554 |
| New Jersey           |            160217    |    40.0757 |    -74.4042 |
| New Mexico           |            166667    |    34.5802 |   -105.996  |
| New York             |            190676    |    40.7127 |    -74.006  |
| North Carolina       |             99624.8  |    35.673  |    -79.0393 |
| North Dakota         |             34500    |    47.6201 |   -100.541  |
| Ohio                 |            136783    |    40.2254 |    -82.6881 |
| Oklahoma             |            160683    |    34.9551 |    -97.2684 |
| Oregon               |             43958.6  |    43.9793 |   -120.737  |
| Pennsylvania         |            168537    |    40.97   |    -77.7279 |
| South Carolina       |            251913    |    33.6874 |    -80.4364 |
| Tennessee            |             59317.4  |    35.773  |    -86.282  |
| Texas                |            223232    |    31.2639 |    -98.5456 |
| Utah                 |             10227.7  |    39.4225 |   -111.714  |
| Vermont              |                 0    |    44.5991 |    -72.5003 |
| Virginia             |            149429    |    37.1232 |    -78.4928 |
| Washington           |            101944    |    38.895  |    -77.0365 |
| West Virginia        |            179794    |    38.4758 |    -80.8408 |
| Wisconsin            |             45876    |    44.4309 |    -89.6885 |
| Wyoming              |             11833.3  |    43.17   |   -107.569  |

---

Below shows the Heap Map representation of the number of customers affected. 
<iframe src="Assets/heatMap.html" width=800 height=600 frameBorder=0></iframe>


## Missingness Analysis

### NMAR Analysis

In this section, we explored the possibility of Non-Missing at Random (NMAR) in our dataset. While no code is provided for this analysis (as NMAR determination involves reasoning about the data generating process), we believe that the column 'CAUSE.CATEGORY.DETAIL' is likely NMAR. This belief stems from the understanding that detailed descriptions of event categories in 'CAUSE.CATEGORY.DETAIL' could result in complex and non-random missingness. Additional data on the uniqueness or complexity of the cause might help make it Missing at Random (MAR).

### Missingness Dependency

Since our question is based around whether outage handling has improved over time, we chose 'OUTAGE.DURATION' for analysis due to its non-trivial missingness and its contribution to measuring improvements in outage handling. Permutation tests were performed to analyze the dependency of the missingness of 'OUTAGE.DURATION' on other columns.

- **Dependency on 'CUSTOMERS.AFFECTED':** [Insert interpretation here based on the test result]
- **Dependency on 'TOTAL.SALES':** [Insert interpretation here based on the test result]

Visualization: The histogram shows the distribution of 'CUSTOMERS.AFFECTED' concerning missingness of 'OUTAGE.DURATION.'

### Hypothesis Testing

We formulated a hypothesis test, with the following components:

- **Null Hypothesis (H0)**: [Insert null hypothesis here]
- **Alternative Hypothesis (H1)**: [Insert alternative hypothesis here]
- **Test Statistic**: [Insert chosen test statistic here]
- **Significance Level**: [Insert chosen significance level here]
- **P-value**: [Insert calculated p-value here]
- **Conclusion**: [Insert conclusion here]

Visualization: Optionally, include a visualization related to the hypothesis test.
Include Optiontional Graph

## Conclusion

Summarize the key findings, limitations, and potential future work.
