# Predictive-alerting-for-cloud-metrics

This repository contains the solution for predictive alerting problem. The model predicts whether an incident will occur within the next H time steps based on the previous W steps of one or more time-series metrics. The sliding-window formulation was used and the model was trained using ??? machine-learning framework.

## Used dataset

For training and evaluating the model I used "realAWSCloudwatch" dataset from the open-source Numenta Anomaly Benchmark (NAB) for the following reasons:
- It uses AWS server metrics, which is exactly the topic of the proposed internship project.
- This dataset contains actual server metrics collected by Amazon CloudWatch, with realistic noise, bursts and drifts.
- This dataset was specifically made for anomaly detection benchmarking and has all anomalies labeled.

The "realAWSCloudwatch" store multiple server metrics (CPU utilization, network bytes in, and disk I/O) as a separate CSV files with simple structure: timestamp (string), value (float).

Incident labels are stored in common for all NAB datasets files in ```labels/``` folder. It map each data file to its anomaly timestamps and anomaly windows. ```combined_labels.json``` lists the exact anomaly timestamps for each series, while ```combined_windows.json``` combine those timestamps into start/end windows that define the time intervals during which an anomaly is considered "active".

## Modeling choices 

The history window W and forecast horizon H tradeoff:
- If W is small, the model sees only very recent behavior, thus, it reacts quickly but becomes sensitive to noise and random spikes 
- If W is large, the model can capture slower trends and pre-incident signs, but recent changes get overlooked 
- If H is small, the model doing very near-term prediction
- If H is large, the model doing early-warning

For cloud incident prediction, we want enough history to see early incident signs (rising latency, growing variance) and a horizon that gives the system enough time to prevent the incident.

## Evaluation setup (including alert thresholds and metrics)

## Result analysis

### Limitations 

### Adaptation a real alerting system 