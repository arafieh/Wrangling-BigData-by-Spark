## Data Engineering for getting inside of Immigrants Destination Choice
### Capstone Project

#### Project Summary
The goal of this project is to create an ETL pipeline using I94 immigration data, city temperature data and U.S. demographics states data to form a database that is optimized for queries on immigration events. This database can be used to answer questions relating immigration behavior to destination demographics and/or temperature. For example do people tend to immigrate to warmer places? or how state divercity does play role on immigrants destination choice.

The project follows the follow steps:
* Step 1: Scope the Project and Gather Data
* Step 2: Wrangling the Data
* Step 3: Define the Data Model
* Step 4: Run ETL to Model the Data
* Step 5: Complete Project Write Up

### Step 1: Scope the Project and Gather Data

#### Scope 
For this project, 3 different data set has been used and pull data from all sources and create fact and dimension tables to show movement of immigration.
I94 immigration data, temperature data and U.S. cities demographics used for dimensional tables. Spark will be used to process the data.

#### Describe and Gather Data 
The I94 immigration [data](https://travel.trade.gov/research/reports/i94/historical/2016.html) comes from the US National Tourism and Trade Office. It is provided in SAS7BDAT [format](https://cran.r-project.org/web/packages/sas7bdat/vignettes/sas7bdat.pdf) which is a binary database storage format. Some relevant attributes include:
* i94yr = 4 digit year
* i94mon = numeric month
* i94cit = 3 digit code of origin city
* i94port = 3 character code of entering USA city
* i94mode = arriving model
* depdate = departure date from the USA
* i94visa = type of  visa

The temperature [data](https://www.kaggle.com/berkeleyearth/climate-change-earth-surface-temperature-data) comes from Kaggle. It is provided in csv format. Some relevant attributes include:
* AverageTemperature = average temperature (Celsius)
* City
* Country
* Latitude
* Longitude 

U.S. City Demographic Data: comes from [OpenSoft](https://public.opendatasoft.com/) and includes data by city, state, age, population, veteran status and race. Some relevant attributes include:
* City
* State
* Total number of population
* Race and their number
* Number of Veterans
* Number of Foreign born

### Step 2: Wrangling the Data

After accessing and exploring on datasets, for each datasets some steps applied and needed columns for data model selected in final step.

#### Cleaning Data

##### 1. Temperature Dataset:
    - Removing null rows
    - Filtering on country by United State
    - Filtering on last year on dataset, 2013
    - Adding state name baseed on city name
    - Selecting needed columns and change their names

##### 2. I94 Immigration Data
    - Removing null rows
    - Validating destination city, port, visa and arriving model
    - Changing arriving port and destination to complete city and state name
    - Selecting needed columns for data model and change names
> **Note**: in some rows arriving port state were different from destination address state. In this case, difference has been assumed is true and passenger select different entering point than final address

##### 3. US Cities Demographic Data:
    - Remove nulls
    - Calculate percentage of population related columns
    - Pivot on `Race` diversity
    - Organize  by state and put average for numeric columns
    - Select needed columns for data model and make final dataset

### Step 3: Define the Data Model

#### Conceptual Data Model
During `Wrangling process`, an schema of data model, based on objective directions of this analysis project, was in mind and so selected columns rely on.
`Star Schema` has been selected to connect fact and dimensional tables.

#### Mapping Out Data Pipelines
For mapping data, following step has been done:
* Dimension tables create from cleans data.
* Fact table created as a SQL query with joins to dimension tables.

### Step 4: Run Pipelines to Model the Data 
#### Data Model and Data Dictionary
Data model, developed based on final datasets on following items:

1. **U.S. Demographic by State**

- state: string (nullable = true)-Full state name
- state_code: string (nullable = true)-Abbreviated state code
- pct_veterans: double (nullable = true)-% Avg Veteran population per state
- pct_foreign_born: double (nullable = true)-% Avg Foreign-Born population per state
- native_american: double (nullable = true)-% Avg Native American population per state
- asian: double (nullable = true)-% Avg Asian population per state
- hispanic_latino: double (nullable = true)% Avg Hispanic or Latino population per state
- black: double (nullable = true)-% Avg Black population per state
- white: double (nullable = true)-% Avg White population per state

2. **Immigration Data by State with Origin**
- year: integer (nullable = true)-Year of immigration
- month: integer (nullable = true)-Month of immigration
- origin_country: string (nullable = true)-Country of origin
- arriving_model: string (nullable = true)-How immigrant entered (Air, Land, Sea)
- visa_type: string (nullable = true)-Type of immigrant visa
- city_port_name: string (nullable = true)-City port name
- state_port_name: string (nullable = true)-State port name
- state_code: string (nullable = true)-Abbreviated destination state code
- dest_state_name: string (nullable = true)-State destination name

3. **Temperature Data by State**

- year: integer (nullable = true)- Temperature Year
- month: integer (nullable = true)- Temperature Month
- avg_temp_celsius: double (nullable = true)- Avg Temperature in Celsius per State
- state_code: string (nullable = true)-Abbreviated State Code
- State: string (nullable = true)-State Name

4. **Fact Table**

- year: integer (nullable = true)-Year from immigration table
- immig_month: integer (nullable = true)-Month from immigration table
- immig_from: string (nullable = true)-Country of Origin from immigration table
- immig_state: string (nullable = true)-State immigrated to from immigration table
- immig_state_count: long (nullable = false)-Total count of people immigrated per state from immigration table
- pct_foreign_born: double (nullable = true)-Avg % foreign born from Demographic table
- native_american: double (nullable = true)-Avg % Native American population from Demographic table
- asian: double (nullable = true)-Avg % Asian population from Demographic table
- hispanic_latino: double (nullable = true)-% Avg Hispanic or Latino population per state from Demographic table
- black: double (nullable = true)-% Avg Black population per state from Demographic table
- white: double (nullable = true)-% Avg White population per state from Demographic table


#### Data Quality Checks
During wrangling process, some part data quality has applied in removing null rows. Now the data quality check will ensure there are adequate number of entries in each table.

### Step 5: Complete Project Write Up
* Clearly state the rationale for the choice of tools and technologies for the project.

Spark was chosen since it can easily handle multiple file formats (including SAS) containing large amounts of data. Spark SQL was chosen to process the large input files into dataframes and manipulated via standard SQL join operations to form additional tables.

* Propose how often the data should be updated and why.

The data should be updated monthly in conjunction with the current raw file format.

* Write a description of how you would approach the problem differently under the following scenarios:

>The data was increased by 100x.
 
If the data was increased by 100x, we would no longer process the data as a single batch job. We could perhaps do incremental updates. We could also consider moving Spark to cluster mode using a cluster manager such as Yarn.  
 
>The data populates a dashboard that must be updated on a daily basis by 7am every day.
 
If the data needs to populate a dashboard daily to meet an SLA then we could use a scheduling tool such as [Airflow](https://airflow.apache.org) to run the ETL pipeline overnight.
 
>The database needed to be accessed by 100+ people.
 
If the database needed to be accessed by 100+ people, we could consider publishing the parquet files to HDFS and giving read access to users that need it. Using cloud services like AWS and Azure are other options