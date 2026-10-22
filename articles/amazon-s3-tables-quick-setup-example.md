# Amazon S3 Tables: Quick Setup Example

- Date: May 27th, 2026
- Category: Technical
- Authors: Carolina Piro; Uriel Muñoz
- URL: https://www.marvik.ai/blog/amazon-s3-tables-quick-setup-example

---

In this post, we provide a simple code example for you to start exploring S3 Tables capabilities in your own AWS account. In this example we will:

- Load data into an S3 Iceberg table leveraging the Amazon S3 Tables Iceberg REST endpoint , via PyIceberg Python library.

- And then query it using Amazon Athena, leveraging the AWS Glue Data Catalog integration .

If you want to know the basics of S3 tables, dive in into its features and use cases, read our previous post of this series: Amazon S3 Tables: The Future of AWS Lakehouses .

## Provision Infrastructure Resources

The first step is to provision and configure the necessary resources in our AWS account. We will connect to the account locally, using the AWS CLI, and manage our resources as infrastructure-as-code, using Terraform.

### Access your AWS account locally

A recommended way to use short-term credentials to connect to your AWS account from your local computer is using the AWS CLI, and updating the config and credentials files on your users’ .aws folder. Using your profile as the default one, your files should look like this:

```
[default]
region = <YOUR_AWS_REGION>
output = json
```

```
[default]
aws_access_key_id=<YOUR_AWS_ACCESS_KEY_ID>
aws_secret_access_key=<YOUR_AWS_SECRET_ACCESS_KEY>
aws_session_token=<YOUR_AWS_SESSION_TOKEN>
```

### Configure your resources

The resources we will create are:

- Two S3 general purpose buckets , one to save the source CSV file and other to use as Athena’s query results location.

- An S3 table bucket , to save our Iceberg table, and a namespace .

- An Athena workgroup , to run the queries.

In this example, we will provision our resources using Terraform, but you can use the AWS Console or CLI instead if you prefer. Our configuration files are the following:

```
# Terraform Configuration ----------------------- 

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }

    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }

  required_version = ">= 1.2"
}

# Cloud Providers Info --------------------------

provider "aws" {
  region = "us-east-1"
}

provider "random" {

}
```

```
data "aws_caller_identity" "current" {
  
}

resource "random_id" "suffix" {
  byte_length = 5
}

# S3 Standard Buckets ----------------------------

# --- Source data files Bucket

resource "aws_s3_bucket" "raw_files_bucket" {
  bucket = "raw-files-bucket-${random_id.suffix.hex}"
}

# --- Athena Query Results Bucket

resource "aws_s3_bucket" "athena_results_bucket" {
  bucket = "athena-results-bucket-${random_id.suffix.hex}"
}

# S3 Tables --------------------------------------

# --- Table Bucket

resource "aws_s3tables_table_bucket" "blog_test_table_bucket" {
  name = "blog-test-table-bucket-${random_id.suffix.hex}"

  # Explicitly define the default encryption to satisfy the provider
  encryption_configuration = {
    sse_algorithm = "AES256"
    kms_key_arn   = null
  }

  maintenance_configuration = {
    iceberg_snapshot_management = {
      status = "enabled"
    }
    iceberg_unreferenced_file_removal = {
      status = "enabled"
      settings = {
        non_current_days  = 7
        unreferenced_days = 7
      }
    }
  }
}

# --- Namespace

resource "aws_s3tables_namespace" "blog_test_namespace" {
  namespace        = "blog_test_namespace"
  table_bucket_arn = aws_s3tables_table_bucket.blog_test_table_bucket.arn
}


# Athena ------------------------------------------

# --- Workgroup

resource "aws_athena_workgroup" "blog_test_athena_workgroup" {
  name = "blog-test-athena-workgroup"

  configuration {
    result_configuration {
      output_location = 
	"s3://${aws_s3_bucket.athena_results_bucket.bucket}/"
    }

    engine_version {
      selected_engine_version = "Athena engine version 3"
    }
  }
}

```

In the Console, you will see your resources in:

- Amazon S3 - Buckets - General purpose buckets

- Amazon S3 - Buckets - Table buckets

- Amazon Athena - Administration - Workgroups

## Load Data into Iceberg Tables

### Data Source

Next, we will generate a CSV file with dummy Orders data, and we will upload it into the S3 general purpose bucket. To do so, we will use the following Python scripts:

```
import csv
import random
from datetime import datetime, timedelta

statuses = ["pending_payment", "paid", "partially_paid"]
start_date = datetime(2026, 3, 1)

with open("<your_file_path>", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["order_id", "order_datetime", 
	              "order_client_id", "order_status"])

    for i in range(1, 1001):
        # Generate random values
        order_time = start_date + 
			 timedelta(hours=i, minutes=random.randint(0, 59))
        client_id = random.randint(1000, 1100)
        status = random.choice(statuses)

        writer.writerow(
            [i, order_time.strftime("%Y-%m-%d %H:%M:%S"), client_id, status]
        )

print("order_data.csv created with 1000 rows.")
```

```
import boto3
import os
from botocore.exceptions import ClientError

def upload_csv_to_s3(local_path, bucket_name, s3_key):
    """
    Uploads a local CSV file to a general purpose S3 bucket. 
    Parameters:
     - local_path: Path to the file on your computer 
(for example: 'data/orders.csv')
     - bucket_name: The name of your S3 bucket
     - s3_key: The destination path in S3
      (for example:  'raw-data/orders.csv')
    """
    # Initialize the S3 client
    s3_client = boto3.client("s3")

    try:
        print(f"Uploading {local_path} to s3://{bucket_name}/{s3_key}...")
        # Perform the upload
        s3_client.upload_file(local_path, bucket_name, s3_key)
        print("Upload Successful!")
        return True

    except FileNotFoundError:
        print(f"The file {local_path} was not found.")
    except ClientError as e:
        print(f"Failed to upload to S3: {e}")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")

    return False


if __name__ == "__main__":
    LOCAL_FILE = "<your_file_path>"
    BUCKET = "<your_raw_data_bucket_name>"
    DESTINATION_KEY = "<your_destination_file_path>"

    upload_csv_to_s3(LOCAL_FILE, BUCKET, DESTINATION_KEY)
```

To upload the file into the S3 bucket, we use the boto3 library, which leverages the AWS credentials configured in the user’s aws files.

In the Console, you will see your resources in:

- Amazon S3 - Buckets - General purpose buckets - <your_raw_data_bucket_name>

### Ingest Data into S3 Tables

To read the CSV data from the general purpose bucket we will use pandas and pyarrow Python libraries. And, as we mentioned, to upload the data into an Iceberg table, we will use the PyIceberg Python library, and we will connect directly to S3 tables using the Amazon S3 Tables Iceberg REST endpoint.

In our Python script, we included the code to optionally create the Iceberg table, using the same Iceberg endpoint.

```
import boto3
import pandas as pd
import pyarrow as pa
from pyiceberg.catalog import load_catalog
from pyiceberg.schema import Schema
from pyiceberg.types import NestedField, IntegerType, StringType, TimestampType

# --- CONFIGURATION ---
REGION = "<YOUR_AWS_REGION>"
TABLE_BUCKET_NAME = "<your_table_bucket_name>" 
NAMESPACE = "blog_test_namespace"
TABLE_NAME = "orders"
SOURCE_CSV_S3_PATH = 
"s3://<your_raw_data_bucket_name>/<your_destination_file_path>"

ORDERS_ICEBERG_SCHEMA = Schema(
    NestedField(1, "order_id", IntegerType()),
    NestedField(2, "order_datetime", TimestampType()),
    NestedField(3, "order_client_id", IntegerType()),
    NestedField(4, "order_status", StringType()),
)

ORDERS_CSV_SCHEMA = {
    "order_id": "Int32",
    "order_client_id": "Int32",
    "order_status": "string",
}
ORDERS_CSV_TIMESTAMPS = ["order_datetime"]
# --- ------------- ---

def get_s3_tables_catalog():
    """Initializes the Iceberg REST catalog for S3 Tables"""
    print("Getting Catalog ...")

    # Dynamically grab credentials from your local AWS session
    session = boto3.Session()
    creds = session.get_credentials().get_frozen_credentials()

    try:
        catalog = load_catalog(
            "s3tablescatalog",
            **{
                "type": "rest",
                "uri": f"https://s3tables.{REGION}.amazonaws.com/iceberg",
                "warehouse": "<your_table_bucket_arn>",
                "rest.sigv4-enabled": "true",
                "rest.signing-name": "s3tables",
                "rest.signing-region": REGION,
                # Explicitly pass the credentials, including the TOKEN
                "s3.access-key-id": creds.access_key,
                "s3.secret-access-key": creds.secret_key,
                "s3.session-token": creds.token,
            },
        )
    except Exception as e:
        raise Exception(f"Failed to load catalog: {e}")

    print("Catalog retrieved")
    return catalog


def load_data_to_iceberg(create_table=False, operation="append"):

    # 1. Initialize Catalog
    catalog = get_s3_tables_catalog()
    table_identifier = f"{NAMESPACE}.{TABLE_NAME}"
    table = None

    # 2. Obtain or Create table
    if create_table:
        print(f"Creating table {table_identifier}...")
        schema = ORDERS_ICEBERG_SCHEMA
        table = catalog.create_table(table_identifier, schema=schema)
        print(f"Table {table_identifier} created.")
    else:
        try:
            table = catalog.load_table(table_identifier)
            print(f"Table {table_identifier} found.")
        except Exception as e:
            print(f"Exception while obtaining the table: {e}")

    # 3. Read CSV from S3 into a Pandas/Arrow Table
    print(f"Reading source data from {SOURCE_CSV_S3_PATH}...")

    df = pd.read_csv(
        	SOURCE_CSV_S3_PATH, 
		dtype=ORDERS_CSV_SCHEMA, 
		parse_dates=ORDERS_CSV_TIMESTAMPS
    )
    arrow_table = pa.Table.from_pandas(df)

    # 4. Append or Overwrite Data
    if operation == "append":
        print(f"Appending {len(df)} rows to {table_identifier}...")
        table.append(arrow_table)
    elif operation == "overwrite":
        print(f"Overwriting {len(df)} rows to {table_identifier}...")
        table.overwrite(arrow_table)
    print("Done! Data is now live in the Lakehouse.")


if __name__ == "__main__":
    load_data_to_iceberg(create_table=False, operation="overwrite")

```

In the Console, you will see your resources in:

- Amazon S3 - Buckets - Table buckets - <your_table_bucket_name>

## Consume Data from Iceberg Tables

### Integration with AWS analytics services

To read the Iceberg data using Athena, we need to Enable Integration with AWS analytics services for the Table buckets in your account. You can do it from the Console in:

- Amazon S3 - Buckets - Table buckets

After this, you can also see the Iceberg table in Glue Data Catalog. You can check from the Console in:

- AWS Glue - Data Catalogs - Catalog

There, you will see a Federated Catalog named s3tablescatalog , with Source = S3 Tables.

### Read the Data in S3 Tables

After integrating with Glue, you can use the Console to query your Iceberg table:

- Go to Amazon Athena - Query Editor.

- In the Data pane, select: Data source: AwsDataCatalog Catalog: s3tablescatalog/<your_table_bucket_name> Database: "blog_test_namespace" or <your_s3tables_namespace_name>

- Data source: AwsDataCatalog

- Catalog: s3tablescatalog/<your_table_bucket_name>

- Database: "blog_test_namespace" or <your_s3tables_namespace_name>

- In the Workgroup dropdown, on the top right, select your workgroup, provisioned at the beginning of the example.

Now you can explore your data using Athena!
