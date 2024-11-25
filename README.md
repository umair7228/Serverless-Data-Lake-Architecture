# Serverless Data Lake Architecture with AWS

This project demonstrates a serverless data lake architecture using various AWS services. The architecture is designed to ingest, process, and analyze CSV files stored in an Amazon S3 bucket. AWS Lambda functions, Glue Crawlers, Glue Jobs, CloudWatch Rules, and SNS topics are orchestrated to automate data processing and send notifications upon completion.

## Project Architecture
![architecture-diagram](https://github.com/user-attachments/assets/999cd237-5b7f-4adc-a81e-6d8634f5a5df)

### Workflow

1. **S3 Bucket**: Stores the source CSV files.
2. **Lambda Trigger**: An S3 event notification triggers a Lambda function upon file upload.
3. **Glue Crawler**: The Lambda function starts a Glue Crawler, which crawls the data in the S3 bucket, creating metadata and storing it in a Glue catalog.
4. **CloudWatch Rule for Crawler Success**: A CloudWatch rule monitors the success state of the Glue Crawler and triggers another Lambda function if the crawler completes successfully.
5. **Lambda to Trigger Glue Job**: The second Lambda function triggers a Glue Job, which processes the data and stores the output in the S3 bucket in Parquet format.
6. **CloudWatch Rule for Job Success**: Another CloudWatch rule monitors the Glue Job's success state, and upon success, triggers an SNS topic.
7. **SNS Notification**: The SNS topic sends an email notification to inform the user that the data processing is complete.

### AWS Services Used

- **S3**: Stores raw and processed data.
- **Lambda**: Automates the process by triggering crawlers and jobs.
- **Glue Crawler**: Creates metadata from the CSV files in S3.
- **Glue Job**: Processes the data and outputs it in Parquet format.
- **CloudWatch**: Monitors the success state of crawlers and jobs.
- **SNS**: Sends notifications on job completion.

### Code Overview

- **Lambda Code to Trigger Glue Crawler**:
    ```python
    import json
    import boto3
    glue = boto3.client('glue')

    def lambda_handler(event, context):
        response = glue.start_crawler(Name='YOUR_CRAWLER_NAME')
        return {
            'statusCode': 200,
            'body': json.dumps('Glue Crawler Started')
        }
    ```

- **Lambda Code to Trigger Glue Job**:
    ```python
    import json
    import boto3
    glue = boto3.client('glue')

    def lambda_handler(event, context):
        response = glue.start_job_run(JobName='YOUR_JOB_NAME')
        return {
            'statusCode': 200,
            'body': json.dumps('Glue Job Started')
        }
    ```

- **Glue Job Script**:
    ```python
    import sys
    from awsglue.transforms import *
    from awsglue.utils import getResolvedOptions
    from pyspark.context import SparkContext
    from awsglue.context import GlueContext
    from awsglue.job import Job

    args = getResolvedOptions(sys.argv, ['JOB_NAME'])
    sc = SparkContext()
    glueContext = GlueContext(sc)
    spark = glueContext.spark_session
    job = Job(glueContext)
    job.init(args['JOB_NAME'], args)

    datasource0 = glueContext.create_dynamic_frame.from_catalog(database="YOUR_DATABASE_NAME", table_name="YOUR_TABLE_NAME")
    datasink4 = glueContext.write_dynamic_frame.from_options(frame=datasource0, connection_type="s3", connection_options={"path": "s3://YOUR_OUTPUT_BUCKET/PATH/"}, format="parquet")
    job.commit()
    ```

- **CloudWatch Rules**:
    - Rule to trigger Lambda on Glue Crawler Success:
      ```json
      {
        "source": ["aws.glue"],
        "detail-type": ["Glue Crawler State Change"],
        "detail": {
          "state": ["SUCCEEDED"],
          "crawlerName": ["YOUR_CRAWLER_NAME"]
        }
      }
      ```
    - Rule to trigger SNS on Glue Job Success:
      ```json
      {
        "source": ["aws.glue"],
        "detail-type": ["Glue Job State Change"],
        "detail": {
          "jobName": ["YOUR_JOB_NAME"],
          "state": ["SUCCEEDED"]
        }
      }
      ```
