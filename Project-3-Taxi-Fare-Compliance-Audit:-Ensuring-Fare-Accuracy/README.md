# Taxi Fare Compliance Audit: Ensuring Fare Accuracy

## Project Description
This project validates taxi fare charges against company pricing policies to detect discrepancies and anomalies. Using trip data, it calculates theoretical fares (base fare + distance/time rates) and compares them to actual fare_amount values.

## Pricing Policy
The fare calculation in this project adheres to the official pricing policy provided by the fictional taxi company stakeholder:
| Component         | Rate  | Applied To           |
|-------------------|-------|----------------------|
| Base Fare         | $3.00 | All trips(flat fee)  |
| Per-Mile Charge   | $2.50 | trip_distance(miles) |
| Per-Minute Charge | $0.50 | duration(minutes)    |
**Industry Reference**:
"Rates align with NYC TLC's yellow taxi base fare ($3.00) and per-mile ($2.50) as of 2023 ([Source](https://www.nyc.gov/site/tlc/passengers/taxi-fare.page))

## Objectives
1. ...

## Repository Structure
| File/Folder                       | Description                                              |
|-----------------------------------|----------------------------------------------------------|
| `/Taxi_Trip_Data_preprocessed.csv`| Dataset used in analysis                                 |
| `/Taxi_Fare_Anlysis.ipynb         | Jupyter notebook files for analysis in python            |
| `/datasets/                       | The folder containing the dataset used                   |
| `/results/                        | Folder containing the results of the analysis            |
| `/README.md`                      | Repository documentation file                            |