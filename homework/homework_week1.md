# Preparation

## Data 1

Download data from this [link](https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz)

Run this command from powershell/bash:
```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5433 \
  --db=ny_taxi \
  --table_name=green_tripdata \
  --url=${URL}
```

## Data 2
Download data from this [link](https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv)

Run this command from powershell/bash:
```bash
URL="https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5433 \
  --db=ny_taxi \
  --table_name=zones \
  --url=${URL}
```
## Question 1. Knowing docker tags

Run the command to get information on Docker:
```bash
docker --help
```

Now run the command to get help on the "docker build" command:
```bash
docker build --help
```

Do the same for "docker run".

Which tag has the following text? - Automatically remove the container when it exits

- --delete  
- --rc  
- --rmc  
- --rm   (this is the answer)

Please note that the correct answer is marked in the list of options.

## Question 2. Understanding docker first run

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed (use `pip list`).

What is version of the package wheel?

- 0.42.0 (this is the answer)
- 1.0.0
- 23.0.1
- 58.1.0

## Question 3. Count records

How many taxi trips were totally made on September 18th 2019?

Tip: started and finished on 2019-09-18.

Remember that lpep_pickup_datetime and lpep_dropoff_datetime columns are in the format timestamp (date and hour+min+sec) and not in date.

- 15767
- 15612 (this is the answer)
- 15859
- 89009

Steps to get the answer:

1. Enter to docker posgresql that ingest with data before, use this command:

```bash
pgcli -h localhost -p 5433 -u root -d ny_taxi
```

2. Run this SQL script:
```sql
SELECT 
  COUNT(*) 
FROM 
  green_tripdata 
WHERE 
  DATE(lpep_pickup_datetime) = '2019-09-18' 
  AND DATE(lpep_dropoff_datetime) = '2019-09-18';
```

3. And the result is 15612

## Question 4. Largest trip for each day

Which was the pick up day with the largest trip distance? Use the pick up time for your calculations.

- 2019-09-18
- 2019-09-16
- 2019-09-26 (this is the answer)
- 2019-09-21

Steps to get the answer:

1. Enter to docker posgresql that ingest with data before, use this command:

```bash
pgcli -h localhost -p 5433 -u root -d ny_taxi
```

2. Run this SQL script:
```sql
SELECT 
  DATE(lpep_pickup_datetime) AS pickup_day, 
  SUM(trip_distance) AS total_distance 
FROM 
  green_tripdata 
GROUP BY 
  pickup_day 
ORDER BY 
  total_distance DESC 
LIMIT 1;
```

3. And the result is "2019-09-26" with total distance is 58759.9400000002

## Question 5. Three biggest pick up Boroughs

Consider `lpep_pickup_datetime` in '2019-09-18' and ignoring Borough has Unknown.

Which were the 3 pick up Boroughs that had a sum of `total_amount` superior to 50000?

- "Brooklyn" "Manhattan" "Queens" (this is the answer)
- "Bronx" "Brooklyn" "Manhattan"
- "Bronx" "Manhattan" "Queens"
- "Brooklyn" "Queens" "Staten Island"

Steps to get the answer:

1. Enter to docker posgresql that ingest with data before, use this command:

```bash
pgcli -h localhost -p 5433 -u root -d ny_taxi
```
2. Run this SQL script:

```sql
SELECT 
  z."Borough", 
  SUM(g.total_amount) AS total_amount 
FROM 
  green_tripdata g 
  JOIN zones z ON g."PULocationID" = z."LocationID" 
WHERE 
  DATE(g.lpep_pickup_datetime) = '2019-09-18' 
  AND z."Borough" != 'Unknown' 
GROUP BY 
  z."Borough" 
HAVING 
  SUM(g.total_amount) > 50000 
ORDER BY 
  total_amount DESC 
LIMIT 3;
```
3. And the result are "Brooklyn" 96333.23999999955, "Manhattan" 92271.29999999912, "Queens" 78671.70999999883

## Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria, which was the drop off zone that had the largest tip? We want the name of the zone, not the id.

Note: it's not a typo, it's tip, not trip.

- Central Park
- Jamaica
- JFK Airport
- Long Island City/Queens Plaza (this is the answer)

Steps to get the answer:

1. Enter to docker posgresql that ingest with data before, use this command:

```bash
pgcli -h localhost -p 5433 -u root -d ny_taxi
```

2. Run this SQL script:
```sql
SELECT 
  z2."Zone" AS dropoff_zone, 
  g.tip_amount AS total_tip 
FROM 
  green_tripdata g 
  JOIN zones z1 ON g."PULocationID" = z1."LocationID" 
  JOIN zones z2 ON g."DOLocationID" = z2."LocationID" 
WHERE 
  z1."Zone" = 'Astoria' 
  AND DATE(g.lpep_pickup_datetime) BETWEEN '2019-09-01' AND '2019-09-30' 
ORDER BY 
  total_tip DESC 
LIMIT 1;
```

3. And the result is "JFK Airport" with a total tip of 62.31

## Question 7. Creating Resources

After updating the `main.tf` and `variable.tf` files run:

```bash
terraform apply
```

Paste the output of this command into the homework submission form.

For installation step, watch this [video.](https://www.youtube.com/watch?v=Y2ux7gq3Z0o&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=12&ab_channel=DataSlinger)

Locate in specific directory:
```bash
/home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp
```

Create new folder "terrademo":
```bash
mkdir terrademo 
cd terrademo
```

Create new folder "keys":
```bash
mkdir keys 
cd keys
```

Create new key file:
```bash
vi my-creds.json
```

Create new file "main.tf" in folder "terrademo":
```bash
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "5.14.0"
    }
  }
}

provider "google" {
  credentials = file("./keys/my-creds.json")
  project = "sublime-iridium-411308"
  region = "us-central1"
}
```

Make sure to Locate in specific directory  /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp
```bash
export GOOGLE_CREDENTIALS='/home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo/keys/my-creds.json' 
echo $GOOGLE_CREDENTIALS
```

Running this command:
```bash
terraform init
```

Open terraform documentation [here.](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket)

Copy line below to main.tf file:

```bash
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "5.14.0"
    }
  }
}

provider "google" {
  credentials = file("./keys/my-creds.json")
  project = "sublime-iridium-411308"
  region = "us-central1"
}

resource "google_storage_bucket" "demo-terra-bucket" {
  name = "auto-expiring-bucket-demo-terra-bucket"
  location = "US"
  force_destroy = true

  lifecycle_rule {
    condition {
      age = 3
    }
    action {
      type = "Delete"
    }
  }

  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}
```

Type this command in directory  /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo:
```bash
terraform plan
```

Type this command again in directory  /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo:
```bash
terraform plan
```

Type this command in directory  /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo:
```bash
terraform apply
```

Type: yes  
And the result is:
```bash
google_storage_bucket.demo-terra-bucket: Creating... 
google_storage_bucket.demo-terra-bucket: Creation complete after 1s [id=auto-expiring-bucket-demo-terra-bucket]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

For destroy existing google bucket of terraform, use this command (don't do this, if you don't want to loss your process):
```bash
terraform destroy
```

Added new script in main.tf:
```bash
resource "google_bigquery_dataset" "demo_dataset" {
  dataset_id = "demo_dataset"
}
```

Type this command
```bash
terraform fmt 
terraform apply
```

And the result is:
```bash
google_bigquery_dataset.demo_dataset: Creating... 
google_bigquery_dataset.demo_dataset: Creation complete after 1s [id=projects/sublime-iridium-411308/datasets/demo_dataset]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Create a new file `variable.tf`:

```bash
variable "credentials" { 
  description = "My Credentials" 
  default = "./keys/my-creds.json" 
}

variable "project" { 
  description = "Project" 
  default = "sublime-iridium-411308" 
}

variable "region" { 
  description = "Region" 
  default = "us-central1" 
}

variable "location" { 
  description = "Project Location" 
  default = "US" 
}

variable "bq_dataset_name" { 
  description = "My BigQuery Dataset Name" 
  default = "demo_dataset" 
}

variable "gcs_bucket_name" { 
  description = "My Storage Bucket Name" 
  default = "auto-expiring-bucket-demo-terra-bucket" 
}

variable "gcs_storage_class" { 
  description = "Bucket Storage Class" 
  default = "STANDARD" 
}
```

Modify main.tf:
```bash
terraform { 
  required_providers { 
    google = { 
      source = "hashicorp/google" 
      version = "5.14.0" 
    } 
  } 
}

provider "google" { 
  credentials = file(var.credentials) 
  project = var.project 
  region = var.region 
}

resource "google_storage_bucket" "demo-terra-bucket" { 
  name = var.gcs_bucket_name 
  location = var.location 
  force_destroy = true

  lifecycle_rule { 
    condition { 
      age = 3 
    } 
    action { 
      type = "Delete" 
    } 
  }

  lifecycle_rule { 
    condition { 
      age = 1 
    } 
    action { 
      type = "AbortIncompleteMultipartUpload" 
    } 
  } 
}

resource "google_bigquery_dataset" "demo_dataset" { 
  dataset_id = var.bq_dataset_name 
}
```

Type these commands:
```bash
terraform destroy
unset ALL_PROXY
echo $GOOGLE_CREDENTIALS
unset GOOGLE_CREDENTIALS
terraform fmt
terraform plan
terraform apply
```

The output should be:
```bash
google_bigquery_dataset.demo_dataset: Creating... 
google_storage_bucket.demo-terra-bucket: Creating... 
google_bigquery_dataset.demo_dataset: Creation complete after 1s [id=projects/sublime-iridium-411308/datasets/demo_dataset] 
google_storage_bucket.demo-terra-bucket: Creation complete after 2s [id=auto-expiring-bucket-demo-terra-bucket]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```