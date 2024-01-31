Preparation:

Data 1:
Download data from this link: https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz

running this command from powershell/bash

URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5433 \
  --db=ny_taxi \
  --table_name=green_tripdata \
  --url=${URL}

Data 2:
Download data from this link: https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv

running this command from powershell/bash

URL="https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5433 \
  --db=ny_taxi \
  --table_name=zones \
  --url=${URL}




Question 1. Knowing docker tags

Run the command to get information on Docker

docker --help

Now run the command to get help on the "docker build" command:

docker build --help

Do the same for "docker run".

Which tag has the following text? - Automatically remove the container when it exits

--delete
--rc
--rmc
--rm (this is the answer)

Question 3. Count records

How many taxi trips were totally made on September 18th 2019?

Tip: started and finished on 2019-09-18.

Remember that lpep_pickup_datetime and lpep_dropoff_datetime columns are in the format timestamp (date and hour+min+sec) and not in date.

15767
15612 (this is the answer)
15859
89009

Step to get the answer:
1. Enter to docker posgresql that ingest with data before, use this command:
pgcli -h localhost -p 5433 -u root -d ny_taxi

2. Run this sql script:
SELECT COUNT(*) 
FROM green_tripdata
WHERE DATE(lpep_pickup_datetime) = '2019-09-18' 
AND DATE(lpep_dropoff_datetime) = '2019-09-18';

3. And the result is 15612

Question 4. Largest trip for each day

Which was the pick up day with the largest trip distance Use the pick up time for your calculations.

2019-09-18
2019-09-16
2019-09-26 (this is the answer)
2019-09-21

Step to get the answer:
1. Enter to docker posgresql that ingest with data before, use this command:
pgcli -h localhost -p 5433 -u root -d ny_taxi

2. Run this sql script:
SELECT DATE(lpep_pickup_datetime) AS pickup_day, SUM(trip_distance) AS total_distance
FROM green_tripdata
GROUP BY pickup_day
ORDER BY total_distance DESC
LIMIT 1;

3. And the result is "2019-09-26" with total distance is 58759.9400000002

Question 5. Three biggest pick up Boroughs

Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough has Unknown

Which were the 3 pick up Boroughs that had a sum of total_amount superior to 50000?

"Brooklyn" "Manhattan" "Queens" (this is the answer)
"Bronx" "Brooklyn" "Manhattan"
"Bronx" "Manhattan" "Queens"
"Brooklyn" "Queens" "Staten Island"

Step to get the answer:
1. Enter to docker posgresql that ingest with data before, use this command:
pgcli -h localhost -p 5433 -u root -d ny_taxi

2. Run this sql script:
SELECT z."Borough", SUM(g.total_amount) AS total_amount
FROM green_tripdata g
JOIN zones z ON g."PULocationID" = z."LocationID"
WHERE DATE(g.lpep_pickup_datetime) = '2019-09-18' AND z."Borough" != 'Unknown'
GROUP BY z."Borough"
HAVING SUM(g.total_amount) > 50000
ORDER BY total_amount DESC
LIMIT 3;

3. And the result are
"Brooklyn"	96333.23999999955
"Manhattan"	92271.29999999912
"Queens"	78671.70999999883

Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria which was the drop off zone that had the largest tip? We want the name of the zone, not the id.

Note: it's not a typo, it's tip , not trip

Central Park
Jamaica
JFK Airport
Long Island City/Queens Plaza (this is the answer)

Step to get the answer:
1. Enter to docker posgresql that ingest with data before, use this command:
pgcli -h localhost -p 5433 -u root -d ny_taxi

2. Run this sql script:
SELECT 
	z2."Zone" AS dropoff_zone,
	g.tip_amount AS total_tip
FROM 
	green_tripdata g
	JOIN zones z1 ON g."PULocationID" = z1."LocationID"
	JOIN zones z2 ON g."DOLocationID" = z2."LocationID"
WHERE
	z1."Zone" = 'Astoria' AND DATE(g.lpep_pickup_datetime) BETWEEN '2019-09-01' AND '2019-09-30'
ORDER BY
	total_tip DESC
LIMIT 1;

3. And the result are
"JFK Airport"	62.31

Question 7. Creating Resources

After updating the main.tf and variable.tf files run:

terraform apply

Paste the output of this command into the homework submission form.

for installation step, watch this video: https://www.youtube.com/watch?v=Y2ux7gq3Z0o&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=12&ab_channel=DataSlinger

Locate in specific directory

/home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp

Create new folder "terrademo"

mkdir terrademo
cd terrademo

Create new folder "keys"

mkdir keys
cd keys

Create new key file

vi my-creds.json

Create new file "main.tf" in folder "terrademo"

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
  project     = "sublime-iridium-411308"
  region      = "us-central1"
}

Make sure to Locate in specific directory
/home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp

export GOOGLE_CREDENTIALS='/home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo/keys/my-creds.json'
echo $GOOGLE_CREDENTIALS

Running this command:
terraform init

Open terraform documentation
https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket

Copy line below to main.tf file

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
  project     = "sublime-iridium-411308"
  region      = "us-central1"
}

resource "google_storage_bucket" "demo-terra-bucket" {
  name          = "auto-expiring-bucket-demo-terra-bucket"
  location      = "US"
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

Type this command in directory /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo:
terraform plan

Type this command again in directory /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo:
terraform plan

Type this command in directory /home/imansubarkahwork/projects/dezoomcamp2024/1_terraform_gcp/terrademo:
terraform apply

type: yes

And the result is:
google_storage_bucket.demo-terra-bucket: Creating...
google_storage_bucket.demo-terra-bucket: Creation complete after 1s [id=auto-expiring-bucket-demo-terra-bucket]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

For destroy existing google bucket of terraform, use this command (don't do this, if you don't want to loss your process):
terraform destroy

Added new script in main.tf:
resource "google_bigquery_dataset" "demo_dataset" {
  dataset_id = "demo_dataset"
}

Type this command
terraform fmt
terraform apply

And the result is:
google_bigquery_dataset.demo_dataset: Creating...
google_bigquery_dataset.demo_dataset: Creation complete after 1s [id=projects/sublime-iridium-411308/datasets/demo_dataset]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Create new file variable.tf:
variable "credentials" {
  description = "My Credentials"
  default     = "./keys/my-creds.json"
  #ex: if you have a directory where this file is called keys with your service account json file
  #saved there as my-creds.json you could use default = "./keys/my-creds.json"
}

variable "project" {
  description = "Project"
  default     = "sublime-iridium-411308"
}

variable "region" {
  description = "Region"
  #Update the below to your desired region
  default = "us-central1"
}

variable "location" {
  description = "Project Location"
  #Update the below to your desired location
  default = "US"
}

variable "bq_dataset_name" {
  description = "My BigQuery Dataset Name"
  #Update the below to what you want your dataset to be called
  default = "demo_dataset"
}

variable "gcs_bucket_name" {
  description = "My Storage Bucket Name"
  #Update the below to a unique bucket name
  default = "auto-expiring-bucket-demo-terra-bucket"
}

variable "gcs_storage_class" {
  description = "Bucket Storage Class"
  default     = "STANDARD"
}

Modified main.tf:
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.14.0"
    }
  }
}

provider "google" {
  credentials = file(var.credentials)
  project     = var.project
  region      = var.region
}

resource "google_storage_bucket" "demo-terra-bucket" {
  name          = var.gcs_bucket_name
  location      = var.location
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

Type this command:
terraform destroy
unset ALL_PROXY
echo $GOOGLE_CREDENTIALS
unset GOOGLE_CREDENTIALS

terraform fmt
terraform plan
terraform apply

google_bigquery_dataset.demo_dataset: Creating...
google_storage_bucket.demo-terra-bucket: Creating...
google_bigquery_dataset.demo_dataset: Creation complete after 1s [id=projects/sublime-iridium-411308/datasets/demo_dataset]
google_storage_bucket.demo-terra-bucket: Creation complete after 2s [id=auto-expiring-bucket-demo-terra-bucket]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.