# Airflow Sensor-Driven Automated Logistics Data Pipeline

## Overview
This project demonstrates an end-to-end Big Data pipeline for automating logistics data processing using Apache Airflow with a Sensor + Scheduling approach. The pipeline processes raw CSV files uploaded to Google Cloud Storage (GCS), runs Hive queries on a Dataproc cluster, and archives the processed files. It ensures periodic checks for new files and fully automates data ingestion, processing, and archiving.

## Features
- **Automated File Detection**: Periodic checks for new files in GCS using Airflow's GCS Sensor.
- **Data Processing**: Raw data is processed and partitioned into a Hive table on Dataproc.
- **File Archiving**: Processed files are moved to an archive bucket for backup.
- **Scalable and Maintainable**: Built on GCP and orchestrated using Apache Airflow for efficient resource management.

## File Structure
```plaintext
sensor_scheduling_dag_project/
├── dags/
│   └── sensor_scheduling_dag.py    # The Airflow DAG script
├── iam/
│   └── required_roles.txt          # IAM roles and permissions needed
├── requirements.txt                # Python dependencies for Airflow
├── README.md                       # Project documentation
```

## Pre-requisites

### Google Cloud Platform (GCP) Setup:
1. **Google Cloud Storage (GCS):**
   - `logistics-raw` bucket: Stores raw input files.
   - `logistics-archive` bucket: Stores processed/archived files.
2. **Dataproc Cluster:**
   - A single-node Dataproc cluster with Hive installed.
3. **IAM Roles:**
   Assign the following roles to the Airflow service account:
   - `roles/dataproc.editor`
   - `roles/storage.objectAdmin`
   - `roles/airflow.admin`
   - `roles/iam.serviceAccountUser`

### Apache Airflow:
- A running Airflow environment with the required Python dependencies installed.

### Python Dependencies:
```bash
pip install -r requirements.txt
```


## Setup Instructions

### 1. Create GCS Buckets
Run the following commands to create the necessary buckets:
```bash
gsutil mb -l us-central1 gs://logistics-raw
gsutil mb -l us-central1 gs://logistics-archive
```

### 2. Set Up Dataproc Cluster
Create a Dataproc cluster with Hive:
```bash
gcloud dataproc clusters create logistics-cluster \
    --region=us-central1 \
    --zone=us-central1-a \
    --single-node \
    --image-version=2.0-debian10 \
    --optional-components=HIVE \
    --scopes=cloud-platform
```

### 3. Assign IAM Roles
Assign the necessary roles to the service account:
```bash
gcloud projects add-iam-policy-binding <PROJECT_ID> \
    --member="serviceAccount:<SERVICE_ACCOUNT>" \
    --role=roles/dataproc.editor

gcloud projects add-iam-policy-binding <PROJECT_ID> \
    --member="serviceAccount:<SERVICE_ACCOUNT>" \
    --role=roles/storage.objectAdmin

gcloud projects add-iam-policy-binding <PROJECT_ID> \
    --member="serviceAccount:<SERVICE_ACCOUNT>" \
    --role=roles/airflow.admin
```

### 4. Deploy the Airflow DAG
Copy the DAG script to the Airflow DAGs folder:
```bash
cp dags/sensor_scheduling_dag.py /path/to/airflow/dags/
```

#### Restart the Airflow scheduler and webserver to apply changes:
```bash
airflow scheduler &
airflow webserver &
```


## Testing the Pipeline

### Upload Test Files
```bash
gsutil cp logistics_2023_09_01.csv gs://logistics-raw/input_data/
```

### Trigger the DAG

The DAG will run automatically based on the schedule.  
Alternatively, you can trigger it manually:
```bash
airflow dags trigger sensor_scheduling_dag
```

### Verify Outputs

- Check the Dataproc logs for Hive query execution.
- Verify that processed files are moved to the `logistics-archive` bucket.
