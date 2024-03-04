# Homework - Workshop 1
Data Ingestion for Data Engineering Zoomcamp 2024

## Question 1: What is the sum of the outputs of the generator for limit = 5?
A: 10.23433234744176  
B: 7.892332347441762  
C: 8.382332347441762  **(this is my answer)**  
D: 9.123332347441762  

**Here is my answer: 8.382332347441762**



## Question 2: What is the 13th number yielded by the generator?  
A: 4.236551275463989  
B: 3.605551275463989  **(this is my answer)**   
C: 2.345551275463989  
D: 5.678551275463989  

**Here is my answer: 3.605551275463989**

## Question 3: Append the 2 generators. After correctly appending the data, calculate the sum of all ages of people.  
A: 353  **(this is my answer)**
B: 365  
C: 378  
D: 390  

**Here is my answer: 353**

## Question 4: Merge the 2 generators using the ID column. Calculate the sum of ages of all the people loaded as described above.  
A: 215  
B: 266  **(this is my answer)**
C: 241  
D: 258  

**Here is my answer: 266**

**Here is my code to find the answer:**

### 1. Use a generator  
Remember the concept of generator? Let's practice using them to futher our understanding of how they work.

Let's define a generator and then run it as practice.

```python
def square_root_generator(limit):
    n = 1
    sum = 0
    while n <= limit:
        answer = n ** 0.5
        sum += answer
        yield answer, sum
        n += 1
        

# Example usage:
limit = 13
generator = square_root_generator(limit)

for sqrt_value in generator:
    print(sqrt_value, sum)
```  

(1.0, 1.0) <built-in function sum>  
(1.4142135623730951, 2.414213562373095) <built-in function sum>  
(1.7320508075688772, 4.146264369941973) <built-in function sum>  
(2.0, 6.146264369941973) <built-in function sum>  
(2.23606797749979, 8.382332347441762) <built-in function sum>  
(2.449489742783178, 10.83182209022494) <built-in function sum>  
(2.6457513110645907, 13.47757340128953) <built-in function sum>  
(2.8284271247461903, 16.30600052603572) <built-in function sum>  
(3.0, 19.30600052603572) <built-in function sum>  
(3.1622776601683795, 22.4682781862041) <built-in function sum>  
(3.3166247903554, 25.7849029765595) <built-in function sum>  
(3.4641016151377544, 29.249004591697254) <built-in function sum>  
(3.605551275463989, 32.854555867161245) <built-in function sum>  

### 2. Append a generator to a table with existing data  
Below you have 2 generators. You will be tasked to load them to duckdb and answer some questions from the data.  
Load the first generator and calculate the sum of ages of all people. Make sure to only load it once.  
Append the second generator to the same table as the first.
After correctly appending the data, calculate the sum of all ages of people.  

```python
def people_1():
    for i in range(1, 6):
        yield {"ID": i, "Name": f"Person_{i}", "Age": 25 + i, "City": "City_A"}

for person in people_1():
    print(person)


def people_2():
    for i in range(3, 9):
        yield {"ID": i, "Name": f"Person_{i}", "Age": 30 + i, "City": "City_B", "Occupation": f"Job_{i}"}


for person in people_2():
    print(person)
```  

{'ID': 1, 'Name': 'Person_1', 'Age': 26, 'City': 'City_A'}  
{'ID': 2, 'Name': 'Person_2', 'Age': 27, 'City': 'City_A'}  
{'ID': 3, 'Name': 'Person_3', 'Age': 28, 'City': 'City_A'}  
{'ID': 4, 'Name': 'Person_4', 'Age': 29, 'City': 'City_A'}  
{'ID': 5, 'Name': 'Person_5', 'Age': 30, 'City': 'City_A'}  
{'ID': 3, 'Name': 'Person_3', 'Age': 33, 'City': 'City_B', 'Occupation': 'Job_3'}  
{'ID': 4, 'Name': 'Person_4', 'Age': 34, 'City': 'City_B', 'Occupation': 'Job_4'}  
{'ID': 5, 'Name': 'Person_5', 'Age': 35, 'City': 'City_B', 'Occupation': 'Job_5'}  
{'ID': 6, 'Name': 'Person_6', 'Age': 36, 'City': 'City_B', 'Occupation': 'Job_6'}  
{'ID': 7, 'Name': 'Person_7', 'Age': 37, 'City': 'City_B', 'Occupation': 'Job_7'}  
{'ID': 8, 'Name': 'Person_8', 'Age': 38, 'City': 'City_B', 'Occupation': 'Job_8'}  

### 3. Merge a generator  
Re-use the generators from Exercise 2.  

A table's primary key needs to be created from the start, so load your data to a new table with primary key ID.  

Load your first generator first, and then load the second one with merge. Since they have overlapping IDs, some of the records from the first load should be replaced by the ones from the second load.  

After loading, you should have a total of 8 records, and ID 3 should have age 33.  

## Solution: First make sure that the following modules are installed:

```python
#Install the dependencies
#%%capture
!pip install dlt[duckdb]
```  

```bash
Requirement already satisfied: dlt[duckdb] in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (0.4.4)
Requirement already satisfied: PyYAML>=5.4.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (6.0.1)
Requirement already satisfied: SQLAlchemy>=1.4.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (2.0.25)
Requirement already satisfied: astunparse>=1.6.3 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (1.6.3)
Requirement already satisfied: click>=7.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (8.1.7)
Requirement already satisfied: fsspec>=2022.4.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (2024.2.0)
Requirement already satisfied: gitpython>=3.1.29 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (3.1.41)
Requirement already satisfied: giturlparse>=0.10.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (0.12.0)
Requirement already satisfied: hexbytes>=0.2.2 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (1.0.0)
Requirement already satisfied: humanize>=4.4.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (4.9.0)
Requirement already satisfied: jsonpath-ng>=1.5.3 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (1.6.1)
Requirement already satisfied: makefun>=1.15.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (1.15.2)
Requirement already satisfied: orjson<=3.9.10,>=3.6.7 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (3.9.10)
Requirement already satisfied: packaging>=21.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (23.1)
Requirement already satisfied: pathvalidate>=2.5.2 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (3.2.0)
Requirement already satisfied: pendulum>=2.1.2 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (3.0.0)
Requirement already satisfied: pytz>=2022.6 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (2023.3.post1)
Requirement already satisfied: requests>=2.26.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (2.31.0)
Requirement already satisfied: requirements-parser>=0.5.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (0.5.0)
Requirement already satisfied: semver>=2.13.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (3.0.2)
Requirement already satisfied: setuptools>=65.6.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (68.2.2)
Requirement already satisfied: simplejson>=3.17.5 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (3.19.2)
Requirement already satisfied: tenacity>=8.0.2 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (8.2.3)
Requirement already satisfied: tomlkit>=0.11.3 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (0.12.3)
Requirement already satisfied: typing-extensions>=4.0.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (4.9.0)
Requirement already satisfied: tzdata>=2022.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (2023.4)
Requirement already satisfied: win-precise-time>=1.4.2 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (1.4.2)
Requirement already satisfied: duckdb<0.10.0,>=0.6.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from dlt[duckdb]) (0.9.2)
Requirement already satisfied: wheel<1.0,>=0.23.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from astunparse>=1.6.3->dlt[duckdb]) (0.41.2)
Requirement already satisfied: six<2.0,>=1.6.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from astunparse>=1.6.3->dlt[duckdb]) (1.16.0)
Requirement already satisfied: colorama in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from click>=7.1->dlt[duckdb]) (0.4.6)
Requirement already satisfied: gitdb<5,>=4.0.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from gitpython>=3.1.29->dlt[duckdb]) (4.0.11)
Requirement already satisfied: ply in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from jsonpath-ng>=1.5.3->dlt[duckdb]) (3.11)
Requirement already satisfied: python-dateutil>=2.6 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from pendulum>=2.1.2->dlt[duckdb]) (2.8.2)
Requirement already satisfied: backports.zoneinfo>=0.2.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from pendulum>=2.1.2->dlt[duckdb]) (0.2.1)
Requirement already satisfied: time-machine>=2.6.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from pendulum>=2.1.2->dlt[duckdb]) (2.13.0)
Requirement already satisfied: importlib-resources>=5.9.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from pendulum>=2.1.2->dlt[duckdb]) (6.1.1)
Requirement already satisfied: charset-normalizer<4,>=2 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from requests>=2.26.0->dlt[duckdb]) (2.0.4)
Requirement already satisfied: idna<4,>=2.5 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from requests>=2.26.0->dlt[duckdb]) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from requests>=2.26.0->dlt[duckdb]) (1.26.18)
Requirement already satisfied: certifi>=2017.4.17 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from requests>=2.26.0->dlt[duckdb]) (2023.11.17)
Requirement already satisfied: types-setuptools>=57.0.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from requirements-parser>=0.5.0->dlt[duckdb]) (69.0.0.20240125)
Requirement already satisfied: greenlet!=0.4.17 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from SQLAlchemy>=1.4.0->dlt[duckdb]) (3.0.1)
Requirement already satisfied: smmap<6,>=3.0.1 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from gitdb<5,>=4.0.1->gitpython>=3.1.29->dlt[duckdb]) (5.0.1)
Requirement already satisfied: zipp>=3.1.0 in c:\users\Subarkah\anaconda3\envs\gpx_env\lib\site-packages (from importlib-resources>=5.9.0->pendulum>=2.1.2->dlt[duckdb]) (3.17.0)
```  

## SOLUTION AS FOLLOWS:

```python
import duckdb 
connection = duckdb.connect()
```  

```python
TABLE_NAME = "Customers"
import dlt

# define the connection to load to.
# We now use duckdb, but you can switch to Bigquery later
generators_pipeline = dlt.pipeline(destination='duckdb', dataset_name='generators')
```  

## MERGING (Q4)
```python
# we can load any generator to a table at the pipeline destnation as follows:
info = generators_pipeline.run(people_1(),
										table_name=TABLE_NAME,
										write_disposition="merge",
                                        primary_key="ID")

# we can load the next generator to the same or to a different table.
info = generators_pipeline.run(people_2(),
										table_name=TABLE_NAME,
										write_disposition="merge",
                                        primary_key="ID")

# the outcome metadata is returned by the load and we can inspect it by printing it.
print(info)
```  

```bash
Pipeline dlt_ipykernel_launcher load step completed in 0.57 seconds
1 load package(s) were loaded to destination duckdb and into dataset generators
The duckdb destination used duckdb:///C:\Users\Subarkah\Documents\CODE\zoomcamp\dlt_ipykernel_launcher.duckdb location to store data
Load package 1707725683.1164558 is LOADED and contains no failed jobs
```  

```python
conn = duckdb.connect(f"{generators_pipeline.pipeline_name}.duckdb")
```  

```python
# let's see the tables
conn.sql(f"SET search_path = '{generators_pipeline.dataset_name}'")
print('Loaded tables: ')
display(conn.sql("show tables"))
```  

```python
Loaded tables: 
┌─────────────────────┐
│        name         │
│       varchar       │
├─────────────────────┤
│ _dlt_loads          │
│ _dlt_pipeline_state │
│ _dlt_version        │
│ customers           │
└─────────────────────┘
```  

```python
# and the data

print(f"{TABLE_NAME} table below:")

all_data = conn.sql(f"SELECT * FROM {TABLE_NAME}").df()
display(all_data)
```  

```python
Customers table below:
id	name	age	city	_dlt_load_id	_dlt_id	occupation
0	1	Person_1	26	City_A	1707725681.9431922	pHZAkq7RZQjQwg	None
1	2	Person_2	27	City_A	1707725681.9431922	iT9WmOV7+wi5/w	None
2	4	Person_4	34	City_B	1707725683.1164558	wsPcl0rC3/HkLg	Job_4
3	3	Person_3	33	City_B	1707725683.1164558	y/C29smLubg6tg	Job_3
4	6	Person_6	36	City_B	1707725683.1164558	me17WTMZPFnJfQ	Job_6
5	5	Person_5	35	City_B	1707725683.1164558	1wycboY1jskYGA	Job_5
6	8	Person_8	38	City_B	1707725683.1164558	w7IxpgomiCrTFg	Job_8
7	7	Person_7	37	City_B	1707725683.1164558	hF2htJrFMVYOcA	Job_7
```  

```python
sum_of_ages = conn.sql(f"SELECT SUM(age) FROM {TABLE_NAME}").df()
display(sum_of_ages)
```  

```python
sum(age)
0	266.0
```  

**Question 4. Merge the 2 generators using the ID column. Calculate the sum of ages of all the people loaded as described above. 
    [266]**

## APPENDING (Q3)

```python
q3_pipeline = dlt.pipeline(destination='duckdb', dataset_name='solving_q3')
conn2 = duckdb.connect(f"{q3_pipeline.pipeline_name}.duckdb")
# we can load any generator to a table at the pipeline destnation as follows:
info = q3_pipeline.run(people_1(),
										table_name=TABLE_NAME,
										write_disposition="append")

# we can load the next generator to the same or to a different table.
info = q3_pipeline.run(people_2(),
										table_name=TABLE_NAME,
										write_disposition="append")

# the outcome metadata is returned by the load and we can inspect it by printing it.
print(info)
```  

```bash
Pipeline dlt_ipykernel_launcher load step completed in 0.22 seconds
1 load package(s) were loaded to destination duckdb and into dataset solving_q3
The duckdb destination used duckdb:///C:\Users\Subarkah\Documents\CODE\zoomcamp\dlt_ipykernel_launcher.duckdb location to store data
Load package 1707725685.39706 is LOADED and contains no failed jobs
```  

```python
# Establishing Connection
conn2.sql(f"SET search_path = '{q3_pipeline.dataset_name}'")
print('Loaded tables: ')
display(conn.sql("show tables"))
```  

```python
Loaded tables: 
┌─────────────────────┐
│        name         │
│       varchar       │
├─────────────────────┤
│ _dlt_loads          │
│ _dlt_pipeline_state │
│ _dlt_version        │
│ customers           │
└─────────────────────┘
```  

```python
# Data
print(f"{TABLE_NAME} table below:")
all_data = conn2.sql(f"SELECT * FROM {TABLE_NAME}").df()
display(all_data)
```  

```python
Customers table below:
id	name	age	city	_dlt_load_id	_dlt_id	occupation
0	1	Person_1	26	City_A	1707725684.4877658	lDs+KzQRdFdIhg	None
1	2	Person_2	27	City_A	1707725684.4877658	Tvo7FAnGoY5NEw	None
2	3	Person_3	28	City_A	1707725684.4877658	m++GgOsLSOJoLA	None
3	4	Person_4	29	City_A	1707725684.4877658	au6NRtoSVFWnVg	None
4	5	Person_5	30	City_A	1707725684.4877658	e/ERJWiRZVJr7g	None
5	3	Person_3	33	City_B	1707725685.39706	7hhRwnh9uATnKg	Job_3
6	4	Person_4	34	City_B	1707725685.39706	Vmr8KUOwjXgLqA	Job_4
7	5	Person_5	35	City_B	1707725685.39706	WSyX1d/IkxfhzA	Job_5
8	6	Person_6	36	City_B	1707725685.39706	SM3InqgOSnBT+w	Job_6
9	7	Person_7	37	City_B	1707725685.39706	j13rnbUf567vag	Job_7
10	8	Person_8	38	City_B	1707725685.39706	jM4U9uTt5V8x1w	Job_8
```  

```python
sum_of_ages = conn2.sql(f"SELECT SUM(age) FROM {TABLE_NAME}").df()
display(sum_of_ages)
```  

```python
sum(age)
0	353.0
```  

**Question 3. Append the 2 generators. After correctly appending the data, calculate the sum of all ages of people. [353]**  

## (Run the following code to delete from both databases...)

```python
# conn.sql(f"DELETE FROM {TABLE_NAME}")
# all_data = conn.sql(f"SELECT * FROM {TABLE_NAME}").df()
# display(all_data)

# conn2.sql(f"DELETE FROM {TABLE_NAME}")
# all_data = conn2.sql(f"SELECT * FROM {TABLE_NAME}").df()
# display(all_data)
```  