//////////////////////////////////////////Download and Install Snowsql from below link/////////////////////////////////////////////////////////////////
https://docs.snowflake.com/en/user-guide/snowsql-install-config.html#installing-snowsql-on-microsoft-windows-using-the-installer

(or)

//////////////////////////////////////Run below curl command from Command Line To Install Snowsql////////////////////////////////////////////////
curl -O https://sfc-repo.snowflakecomputing.com/snowsql/bootstrap/1.2/windows_x86_64/snowsql-1.2.23-windows_x86_64.msi

(or)

/////////////////////////////////////////Documentation Link To Download///////////////////////////////////////////////////////////////////////////////
https://docs.snowflake.com/en/user-guide/snowsql-install-config.html#installing-snowsql-on-microsoft-windows-using-the-installer

///////////////////////////////////////////////Login to snowflake from Snowsql client///////////////////////////////////////////////////////////////////////////////
snowsql -a VYMPWLS-US68401;(Enter user name and passowrd after pressing enter)

(or)

snowsql -a VYMPWLS-US68401 -u sg03snowflake;(Enter passowrd after pressing enter)

(or)

snowsql--snowsql -c sg03snowflake_con;(just press enter)

/////////////////////////////////////////////////////////////Run queries To check data in tables available in 'SNOWFLAKE_SAMPLE_DATA' database/////////////////////////////////

USE WAREHOUSE COMPUTE_WH;

USE DATABASE SNOWFLAKE_SAMPLE_DATA;

USE SCHEMA TPCH_SF1;

SELECT * FROM CUSTOMER;

(or)

SELECT * FROM COMPUTE_WH.SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.CUSTOMER;

/////////////////////////////////////////////////LOAD DATA IN USER INTERNAL STAGE/////////////////////////////////////////////////////////////////////////////////////////////
//Put your files into user internal stage.If your put command is failing with 403 forbidden error, practice this session after AWS-Snowflake integration session(next video in the play list) then it will work.
put file://C:\Users\Shivam_Gupta\Snowflake_Training\customer_data_user.csv @~/staged;
put file://C:\Users\Shivam_Gupta\Snowflake_Training\customer_data_user.csv @~/staged;

//list all the files in user internal stage
list @~/staged;

///////////////////////////////////////////LOAD DATA IN TABLE INTERNAL STAGE//////////////////////////////////////////////////////////////////////////////////
//Put your files into table internal stage
put file://C:\Users\Shivam_Gupta\Snowflake_Training\customer_data_table.csv @%customer_data_table;error :there is no such table named "customer_data_table" so unable to create table internal stage named "customer_data_table"

//Create customer_data_table to load files to table internal stages
CREATE OR REPLACE TABLE mydb.public.customer_data_table (
customerid NUMBER,
custname STRING,
email STRING,
city STRING,
state STRING,
DOB DATE
);

//Now put  your file into table internal stage .it will loaded successfully now
put file://C:\Users\Shivam_Gupta\Snowflake_Training\customer_data_table.csv @%customer_data_table;

//show the list of files in table internal stage
list @%customer_data_table;

////////////////////////////////////////////////LOAD DATA IN NAMED INTERNAL STAGE///////////////////////////////////////////////////////////////////////////////
// Create a schema for internal stages.we will store all stage object in schema named 'internal_stages'.internal satge is database object & is created in same manner as we create external stage
CREATE SCHEMA IF NOT EXISTS mydb.internal_stages

//Create a named stage[3 in qty]
CREATE OR REPLACE STAGE mydb.internal_stages.named_customer_stage;

CREATE OR REPLACE STAGE mydb.internal_stages.named_orders_stage;

CREATE OR REPLACE STAGE mydb.internal_stages.named_product_stage;

//showing all named internal stages
show stages in mydb.internal_stages;

//Put your files into named internal stage
put file://C:\Users\Shivam_Gupta\Snowflake_Training\customer_data_named.csv @mydb.internal_stages.named_customer_stage;

//list all the files in named internal stage
list @mydb.internal_stages.named_customer_stage;

/////////////////////////////////////////////////Load all files data to the table//////////////////////////////////////////////////////////
//Copy all these files to table customer_data_table

COPY INTO mydb.public.customer_data_table
FROM @~/staged/customer_data_user.csv
file_format = (type = csv field_delimiter = '|' skip_header = 1);

COPY INTO mydb.public.customer_data_table
FROM @%customer_data_table/customer_data_table.csv
file_format = (type = csv field_delimiter = '|' skip_header = 1);

COPY INTO mydb.public.customer_data_table
FROM @mydb.internal_stages.named_customer_stage/customer_data_named.csv
file_format = (type = csv field_delimiter = '|' skip_header = 1);

////////////////////////////////////////////////////////Validate the data/////////////////////////////////////////////////////////////////////////////////////
SELECT * FROM mydb.public.customer_data_table;