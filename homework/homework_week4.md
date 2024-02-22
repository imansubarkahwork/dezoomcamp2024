# Question 1.
You'll need to have completed the "Build the first dbt models" video.

What happens when we execute dbt build --vars '{'is_test_run':'true'}':

- It's the same as running dbt build
- It applies a limit 100 to all of our models  
- It applies a limit 100 only to our staging models  
- Nothing  

**Answer: It applies a limit 100 only to our staging models**

# Question 2.
What is the code that our CI job will run?

- The code that has been merged into the main branch  
- The code that is behind the object on the dbt_cloud_pr_ schema  
- The code from any development branch that has been opened based on main  
- The code from a development branch requesting a merge to main  

**Answer: The code from a development branch requesting a merge to main**  

# Question 3.
Setup:  
Create a staging model for the fhv data, similar to the ones made for yellow and green data. Add an additional filter for keeping only records with pickup time in year 2019. Do not add a deduplication step. Run this models without limits (is_test_run: false).

Create a core model similar to fact trips, but selecting from stg_fhv_tripdata and joining with dim_zones. Similar to what we've done in fact_trips, keep only records with known pickup and dropoff locations entries for pickup and dropoff locations. Run the dbt model without limits (is_test_run: false).

What is the count of records in the model fact_fhv_trips after running all dependencies with the test run variable disabled (false)?

- 12998722  
- 22998722  
- 32998722  
- 42998722  

**Answer: 22998722**  

![Alt text](image-30.png)  

## Model:  

```sql
{{ config(materialized='table') }}

SELECT
    pickup_datetime,
    dropOff_datetime AS dropoff_datetime,
    PUlocationID,
    zone1.Borough AS pickup_borough,
    zone1.Zone AS pickup_zone,
    zone1.service_zone AS pickup_service_zone,
    DOlocationID,
    zone2.Borough AS dropoff_borough,
    zone2.Zone AS dropoff_zone,
    zone2.service_zone AS dropoff_service_zone
FROM 
    {{ ref('staging_fhv') }} AS staging_fhv
    INNER JOIN
    {{ ref('zone_dimension') }} AS zone1
    ON
    staging_fhv.PUlocationID = zone1.LocationID
    INNER JOIN
    {{ ref('zone_dimension') }} AS zone2
    ON
    staging_fhv.DOlocationID = zone2.LocationID

{%- if var("is_test_run", "true") %}
LIMIT 1000
{% endif %}
```  

# Question 4.
Create a dashboard with some tiles that you find interesting to explore the data. One tile should show the amount of trips per month, as done in the videos for fact_trips, including the fact_fhv_trips data.  

What is the service that had the most rides during the month of July 2019?  

- FHV  
- Green  
- Yellow  
- FHV and Green  

**Answer: Yellow**

![Alt text](image-31.png)  