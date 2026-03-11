# Predictive-alerting-for-cloud-metrics

This repository contains the solution for predictive alerting problem. The model predicts whether an incident will occur within the next H time steps based on the previous W steps of one or more time-series metrics. The sliding-window formulation was used and the model was trained using ??? machine-learning framework.

## Used dataset

For training and evaluating the model I used "realAWSCloudwatch" dataset from the open-source Numenta Anomaly Benchmark (NAB).
Source: https://github.com/numenta/NAB
Original NAB license and attribution apply to dataset files copied from that repository.

This dataset was chosen for the following reasons:
- It uses AWS server metrics, which is exactly the topic of the proposed internship project.
- This dataset contains actual server metrics collected by Amazon CloudWatch, with realistic noise, bursts and drifts.
- This dataset was specifically made for anomaly detection benchmarking and has all anomalies labeled.

The "realAWSCloudwatch" store multiple server metrics (CPU utilization, network bytes in, and disk I/O) as a separate CSV files with simple structure: timestamp (string), value (float).

Incident labels are stored in common for all NAB datasets files in ```labels/``` folder. It map each data file to its anomaly timestamps and anomaly windows. ```combined_labels.json``` lists the exact anomaly timestamps for each anomaly, while ```combined_windows.json``` store expanded anomaly windows for purposes of NAB scoring, which rewards early detection.

## Modeling choices 

### Data preparation

From metrics that the NAB "realAWSCloudwatch" dataset contains, only CPU utilization files from "ec2" was chosen, to keep data homogeneous. 

Labels for specific anomalies timestamps was taken from ```combined_labels.json```, because anomaly windows provided in ```combined_windows.json``` primarily serve to reward early-warning in NAB scoring. 

The csv files with ```timestamp (string)```, ```value (float)``` were imported in separate dataframes, while converting timestamps to datetime format for simpler future handling and plotting. Then the anomaly timesteps was taken from ```combined_labels.json``` and ```anomaly_label```: ```True``` was added to all anomalies timestamps in dataframes. 

The following plot shows CPU utilization in time for all 8 csv files used with anomalies marked as red dots.
 
![](/pics/cpu_utilization_plot.png)

From obtained graphs we can see that even the data from similar CPU clusters looks quite different. It also could be seen that peaks in CPU utilization aren't necessary an anomaly, but non-standard spikes are (e.g. CPU utilization 1, 3 and 7). 

**[TO-DO]** analyze day/night or other cycles  

The number of incidents is very small in comparison with the size of the dataset. There are approximately 1-3 incidents per 4000 normal values. That means that we need to deal with class imbalance in the process of training. 

### Sliding window approach

We will use a sliding-window to slice our dataset for training the model, where history window W is the number of observations that the model sees to predict if there is the anomaly in the following H observations called forecast horizon.

Dealing with slide window formulation we encounter the history window W and forecast horizon H tradeoff:
- If W is small, the model sees only very recent behavior, thus, it reacts quickly but becomes sensitive to noise and random spikes 
- If W is large, the model can capture slower trends and pre-incident signs, but recent changes get overlooked 
- If H is small, the model doing very near-term prediction
- If H is large, the model doing early-warning

For cloud incident prediction, we want enough history to see early incident signs (rising latency, growing variance) and a horizon that gives the system enough time to prevent the incident.

For the beginning W = 30 (150 minutes) and H = 5 (25 minutes) was chosen arbitrary to tune them in future as hyperparameters. 

### Model choice

RNN:
LSMT

[Why do tree-based models still outperform deep
learning on tabular data?](https://arxiv.org/pdf/2207.08815)

Decision trees:
Random Forest
XGBoost
LightGBM

### Feature engineering  

We will create 5 feature configurations to compare them in evaluation:
1. Raw data as a baseline for our engineered features (LSMT focused).
2. Light feature set with raw data included (LSMT focused).
3. Full feature set with some raw data (Decision trees focused).
4. All features without raw data (Decision trees focused).
5. Subset of statistical features (Decision trees focused). 

This paper showed that extracting statistical features from time series data could work well with decision trees like XGBoost. Despite the fact that the paper focused on on solving challenges with irregular data and missing observation, its results shows that we can achieve high accuracy with a feature set containing just several statistical valuables.
[A Statistical Approach for Modeling Irregular Multivariate Time Series with Missing Observations](https://arxiv.org/html/2602.19531v1)

We use statistical features from this paper: 
1. Mean of observed values.
2. Standard Deviation of Observed Values.
3. Mean Change In Values.
4. Standard Deviation of Change In Values.

Additional statistical features:
1. Minimum of observed values.
2. Maximum of observed values.
3. Rolling mean with window size of 5. 

Engineered features:
1. Linear trend slope fitted over observed values.
2. Hour-of-day encoded as $sin(2\pi h/24)$ and $cos(2\pi h/24)$.
3. Difference between the last value and the mean of observed values, to capture rapid changes.

From these features we form our feature configurations.

#### 1. Raw data (baseline)
1. raw W values

#### 2. Balanced light feature set

#### 3. Full feature set

#### 4. Only features set 

#### 5. Statistical features from the paper
1. Mean of observed values.
2. Standard Deviation of Observed Values.
3. Mean Change In Values.
4. Standard Deviation of Change In Values.

## Evaluation setup (including alert thresholds and metrics)

## Result analysis

## Limitations 

## Adaptation a real alerting system 