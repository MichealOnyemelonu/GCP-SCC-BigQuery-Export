# Export and Analyze Security Command Center Findings with BigQuery

This project demonstrates how to export findings from Google Cloud's Security Command Center (SCC) to BigQuery for analytical purposes. Exporting SCC findings to BigQuery allows organizations to build dashboards and perform detailed analysis on security findings to better understand their security posture.

## Table of Contents
- [Overview](#overview)
- [Setup Instructions](#setup-instructions)
- [Creating a BigQuery Dataset](#creating-a-bigquery-dataset)
- [Configuring Continuous Export from SCC to BigQuery](#configuring-continuous-export-from-scc-to-bigquery)
- [Generating and Analyzing Findings](#generating-and-analyzing-findings)
- [Exporting Existing Findings to BigQuery](#exporting-existing-findings-to-bigquery)
- [Conclusion](#conclusion)
- [References](#references)

## Overview
In this project, we configure continuous exports of SCC findings to a BigQuery dataset. This allows for the creation of analytical dashboards and in-depth queries to better understand security issues within an organization. Additionally, we demonstrate how to export existing findings from SCC to BigQuery via a Google Cloud Storage (GCS) bucket.

## Setup Instructions

### Prerequisites
- A Google Cloud project with Security Command Center and BigQuery enabled.
- Basic knowledge of GCP services, particularly SCC, BigQuery, and Cloud Storage.

### Creating a BigQuery Dataset

1. Open a Cloud Shell session in the Google Cloud Console.
2. Run the following command to create a new BigQuery dataset:
    ```bash
    PROJECT_ID=$(gcloud config get project)
    bq --location=YOUR-LAB-REGION --apilog=/dev/null mk --dataset \
    $PROJECT_ID:continuous_export_dataset
    ```

### Configuring Continuous Export from SCC to BigQuery

1. Enable the SCC service in your project:
    ```bash
    gcloud services enable securitycenter.googleapis.com
    ```
2. Create a new continuous export to BigQuery:
    ```bash
    gcloud scc bqexports create scc-bq-cont-export --dataset=projects/$PROJECT_ID/datasets/continuous_export_dataset --project=$PROJECT_ID
    ```
3. Ensure that you receive a confirmation message similar to:
    ```plaintext
    Created.
    dataset: projects/$PROJECT_ID/datasets/continuous_export_dataset
    principal: service-org-123456789012@gcp-sa-scc-notification.iam.gserviceaccount.com
    ```

### Generating and Analyzing Findings

1. Create three new service accounts and generate keys for them:
    ```bash
    for i in {0..2}; do
      gcloud iam service-accounts create sccp-test-sa-$i;
      gcloud iam service-accounts keys create /tmp/sa-key-$i.json \
      --iam-account=sccp-test-sa-$i@$PROJECT_ID.iam.gserviceaccount.com;
    done
    ```
2. Fetch and analyze the newly created findings in BigQuery:
    ```bash
    bq query --apilog=/dev/null --use_legacy_sql=false  \
    "SELECT finding_id, event_time, finding.category FROM continuous_export_dataset.findings"
    ```
    - Note: It may take a few minutes for the findings to appear.

### Exporting Existing Findings to BigQuery

1. **Create a GCS Bucket**:
   - Navigate to **Cloud Storage > Buckets** in the Cloud Console.
   - Create a new bucket with a unique name, e.g., `scc-export-bucket-$PROJECT_ID`.
   - Set the location type to `Region` and select your lab region.

2. **Export Findings to GCS**:
   - Navigate to **Security > Security Command Center > Findings**.
   - Click the **Export** button, choose **Cloud Storage**, and select the bucket created above.
   - Set the export filename to `findings.jsonl` and choose `JSONL` as the format.
   - Change the time range to `All time` and export the findings.

3. **Load Findings into BigQuery**:
   - Navigate to **BigQuery > BigQuery Studio**.
   - Click on the `+ADD` button and select **Google Cloud Storage** as the data source.
   - Provide the GCS path to `scc-export-bucket-$PROJECT_ID/findings.jsonl`.
   - Use the following schema to create the table:
     ```json
     [
       {
         "mode": "NULLABLE",
         "name": "resource",
         "type": "JSON"
       },
       {
         "mode": "NULLABLE",
         "name": "finding",
         "type": "JSON"
       }
     ]
     ```
   - Name the table `old_findings` and click **CREATE TABLE**.

4. **Preview Findings**:
   - Once the table is created, preview the data to ensure the findings were correctly imported.

### Conclusion

This project demonstrates how to export and analyze SCC findings using BigQuery. By automating the export process and utilizing BigQuery's powerful analytics capabilities, organizations can gain deeper insights into their security posture.

### References
- [Google Cloud Security Command Center Documentation](https://cloud.google.com/security-command-center/docs)
- [Google Cloud BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Google Cloud Storage Documentation](https://cloud.google.com/storage/docs)

