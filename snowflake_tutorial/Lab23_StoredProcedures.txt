// Creating AWS free account
https://www.youtube.com/watch?v=P7hVdusJF7I

//Create a S3 bucket service/resource named 'sg.aws' in aws & load some file in the bucket specific to particular pattern like 'csv','json'...

//Create a Role in aws using 'IAM' service available in aws named 'asw_int' & grant specific permission in that role's to access the s3 bucket by this role & finally make 2 way authentication b/w snowflake & aws by using 'ARM' properties in that role's trust relationship
-----------------------------------------------------------------------------------------------------------------------------------------------
// // Create a storage integration object in Snowflake to access the privately created bucket in aws
create or replace storage integration aws_s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE 
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::767397752598:role/aws_int'
  STORAGE_ALLOWED_LOCATIONS = ('s3://sg.aws-s3/csv/', 's3://sg.aws-s3/json/')
  COMMENT = 'Integration with aws s3 buckets' ;
   
   
// exchange ARN from both side|Get external_id from snowflake side| ARN given by our storage integration object 'aws_s3_int'(by executing below query) and update it in S3 trust & relationship' ARN property & we are done with integration now
DESC integration aws_s3_int;

// ARN -- Amazon Resource Names
// S3 -- Simple Storage Service
// IAM -- Integrated Account Management

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
// Now we are going to extract data available in that s3 bucket 'sg.aws-s3' & load it into snowflake table 'aws_customer_data'|Create database and schema
CREATE DATABASE IF NOT EXISTS MYDB;
CREATE SCHEMA IF NOT EXISTS MYDB.file_formats;

// Create file format object
CREATE OR REPLACE file format mydb.file_formats.csv_fileformat
    type = csv
    field_delimiter = '|'
    skip_header = 1
    empty_field_as_null = TRUE;    
    
// Create stage object with integration object & file format object
CREATE OR REPLACE STAGE mydb.external_stages.aws_s3_csv
    URL = 's3://sg.aws-s3/csv/'
    STORAGE_INTEGRATION = aws_s3_int
    FILE_FORMAT = mydb.file_formats.csv_fileformat ;

//Listing files under your s3 buckets
list @mydb.external_stages.aws_s3_csv;

// Create a table first
CREATE OR REPLACE TABLE mydb.public.aws_customer_data (
customerid NUMBER,
custname STRING,
email STRING,
city STRING,
state STRING,
DOB DATE
); 

// Use Copy command to load the files
COPY INTO mydb.public.aws_customer_data
    FROM @mydb.external_stages.aws_s3_csv
    PATTERN = '.*customer.*';    
	
//Validate the data
SELECT * FROM mydb.public.aws_customer_data;

---------------------------------------------------------------------------------------------------------------------------------
Steps to Load data from Azure

Step 1: Create storage integration between Snowflake and Azure:
https://docs.snowflake.com/en/user-guide/data-load-azure-config.html 

Step 2: Create External Stage objects:
https://docs.snowflake.com/en/user-guide/data-load-azure-create-stage.html 

Step 3: Copy command to load the data from Azure containers to Snowflake tables:
https://docs.snowflake.com/en/user-guide/data-load-azure-copy.html 

-------------------------------------------------------------------------------------------------------------------------------------------------------------
Steps to Load data from GCP

Step 1: Create storage integration between Snowflake and GCP:
https://docs.snowflake.com/en/user-guide/data-load-gcs-config.html 

Step 2: Create External Stage objects:
https://docs.snowflake.com/en/user-guide/data-load-gcs-config.html#step-4-create-an-external-stage

Step 3: Copy command to load the data from Google cloud storage to Snowflake tables:
https://docs.snowflake.com/en/user-guide/data-load-gcs-copy.html 
