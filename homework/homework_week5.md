# Homework 5: Batch processing

## Question 1. Spark version (1 point)

**My answer is 3.3.2**

```python
import pyspark
from pyspark.sql import SparkSession
```  

```python
spark = SparkSession.builder \
    .master("local[*]") \
    .appName('test') \
    .getOrCreate()
```  

```python
spark.version
```  

```python
'3.3.2'
```  

## Question 2. FHV October 2019 partition size (1 point)
1MB  
6MB **(this is my answer)**  
25MB  
87MB  

Read the October 2019 FHV into a Spark Dataframe with a schema as we did in the lessons.

Repartition the Dataframe to 6 partitions and save it to parquet.

What is the average size of the Parquet (ending with .parquet extension) Files that were created (in MB)? Select the answer which most closely matches.

```bash
!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/fhv/fhv_tripdata_2019-10.csv.gz
```  

```bash
!zcat fhv_tripdata_2019-10.csv.gz | head -n 11 
```  

```python
from pyspark.sql import types

filename = 'fhv_tripdata_2019-10.csv.gz'

schema = types.StructType([
    types.StructField('dispatching_base_num', types.StringType(), True),
    types.StructField('pickup_datetime', types.TimestampType(), True),
    types.StructField('dropOff_datetime', types.TimestampType(), True),
    types.StructField('PUlocationID', types.IntegerType(), True),
    types.StructField('DOlocationID', types.IntegerType(), True),
    types.StructField('SR_Flag', types.StringType(), True),
    types.StructField('Affiliated_base_number', types.StringType(), True),
])

df_fhv = spark.read \
    .option("header", "true") \
    .schema(schema) \
    .csv(filename)

df_fhv = df_fhv.repartition(6)

df_fhv.write.parquet('fhv/2019/10/')
```  

```bash
!ls -ltrh fhv/2019/10/*parquet

-rw-r--r-- 1 subarkah subarkah 6.4M Mar  2 04:49 fhv/2019/10/part-00003-b4a20026-8748-4b64-9432-5cb3cd4ca146-c000.snappy.parquet
-rw-r--r-- 1 subarkah subarkah 6.4M Mar  2 04:49 fhv/2019/10/part-00001-b4a20026-8748-4b64-9432-5cb3cd4ca146-c000.snappy.parquet
-rw-r--r-- 1 subarkah subarkah 6.4M Mar  2 04:49 fhv/2019/10/part-00000-b4a20026-8748-4b64-9432-5cb3cd4ca146-c000.snappy.parquet
-rw-r--r-- 1 subarkah subarkah 6.4M Mar  2 04:49 fhv/2019/10/part-00002-b4a20026-8748-4b64-9432-5cb3cd4ca146-c000.snappy.parquet
-rw-r--r-- 1 subarkah subarkah 6.4M Mar  2 04:49 fhv/2019/10/part-00004-b4a20026-8748-4b64-9432-5cb3cd4ca146-c000.snappy.parquet
-rw-r--r-- 1 subarkah subarkah 6.4M Mar  2 04:49 fhv/2019/10/part-00005-b4a20026-8748-4b64-9432-5cb3cd4ca146-c000.snappy.parquet
```  

## Question 3. Count records on 15th of October (1 point)
108,164  
12,856  
452,470  
62,610 **(this is my answer)**  

```python
from pyspark.sql import functions as F

# convert pickup_datetime column to a Date type
df_fhv = df_fhv.withColumn('pickup_date', F.to_date(df_fhv.pickup_datetime))

# filter on the 2019-10-15 + count
df_fhv.filter(df_fhv.pickup_date == "2019-10-15").count()
``` 

```python
62610
``` 

## Question 4. The longest trip (1 point)
631,152.50 Hours **(this is my answer)**  
243.44 Hours  
7.68 Hours  
3.32 Hours  

```python
timeFmt = "yyyy-MM-dd HH24:mm:ss"

timeDiff = (F.unix_timestamp('dropOff_datetime', format=timeFmt)
            - F.unix_timestamp('pickup_datetime', format=timeFmt))

df_fhv.withColumn("duration", timeDiff).agg({"duration": "max"}).collect()[0][0] / 3600
```  

```python
631152.5
```  

## Question 5. Spark UI port (1 point)
80  
443  
4040 **(this is my answer)**  
8080  

## Question 6. Least frequent pickup location zone (1 point)
East Chelsea  
Jamaica Bay **(this is my answer)**  
Union Sq  
Crown Heights North  

```bash
!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```  

```python
!head taxi_zone_lookup.csv
```  

```python
df_zones = spark.read \
    .option("header", "true") \
    .csv('taxi_zone_lookup.csv')

df_zones.show()

+----------+-------------+--------------------+------------+
|LocationID|      Borough|                Zone|service_zone|
+----------+-------------+--------------------+------------+
|         1|          EWR|      Newark Airport|         EWR|
|         2|       Queens|         Jamaica Bay|   Boro Zone|
|         3|        Bronx|Allerton/Pelham G...|   Boro Zone|
|         4|    Manhattan|       Alphabet City| Yellow Zone|
|         5|Staten Island|       Arden Heights|   Boro Zone|
|         6|Staten Island|Arrochar/Fort Wad...|   Boro Zone|
|         7|       Queens|             Astoria|   Boro Zone|
|         8|       Queens|        Astoria Park|   Boro Zone|
|         9|       Queens|          Auburndale|   Boro Zone|
|        10|       Queens|        Baisley Park|   Boro Zone|
|        11|     Brooklyn|          Bath Beach|   Boro Zone|
|        12|    Manhattan|        Battery Park| Yellow Zone|
|        13|    Manhattan|   Battery Park City| Yellow Zone|
|        14|     Brooklyn|           Bay Ridge|   Boro Zone|
|        15|       Queens|Bay Terrace/Fort ...|   Boro Zone|
|        16|       Queens|             Bayside|   Boro Zone|
|        17|     Brooklyn|             Bedford|   Boro Zone|
|        18|        Bronx|        Bedford Park|   Boro Zone|
|        19|       Queens|           Bellerose|   Boro Zone|
|        20|        Bronx|             Belmont|   Boro Zone|
+----------+-------------+--------------------+------------+
only showing top 20 rows
```  

```python
df_zones.write.parquet('zones')
```  

```python
df_zones = spark.read.parquet('zones/')
df_result = df_fhv.join(df_zones, df_fhv.PUlocationID == df_zones.LocationID)
```  

```python
# Counting occurrences of each zone
zone_counts = df_result.groupBy("Zone").count()
```  

```python
# Finding the least frequent pickup location zone
zone_counts.orderBy("count").show()
```  

```python
+--------------------+-----+
|                Zone|count|
+--------------------+-----+
|         Jamaica Bay|    1|
|Governor's Island...|    2|
| Green-Wood Cemetery|    5|
|       Broad Channel|    8|
|     Highbridge Park|   14|
|        Battery Park|   15|
|Saint Michaels Ce...|   23|
|Breezy Point/Fort...|   25|
|Marine Park/Floyd...|   26|
|        Astoria Park|   29|
|    Inwood Hill Park|   39|
|       Willets Point|   47|
|Forest Park/Highl...|   53|
|  Brooklyn Navy Yard|   57|
|        Crotona Park|   62|
|        Country Club|   77|
|     Freshkills Park|   89|
|       Prospect Park|   98|
|     Columbia Street|  105|
|  South Williamsburg|  110|
+--------------------+-----+
only showing top 20 rows
```