# Batch-data-pipeline-using-Airflow-pySpark-EMR-Snowflake

# Airflow EMR Integration

## Overview

This project aims to automate data processing workflows using Apache Airflow, integrating with Amazon EMR for data processing and Snowflake for data warehousing. The project includes setup instructions for the required environments and configurations, as well as SQL queries for creating tables in Snowflake.


<img src="Architecture.png" alt="Architecture Diagram">


## Setup Instructions

### Prerequisites
- Python 3 and pip
- SQLite3
- PostgreSQL
- AWS CLI
- Boto3
- Snowflake Connector

### Installation Steps
```bash
sudo apt update
sudo apt install -y python3-pip
sudo apt install -y sqlite3
sudo apt-get install -y libpq-dev
pip3 install --upgrade awscli
pip3 install boto3
sudo pip3 install virtualenv 
virtualenv venv 
source venv/bin/activate
pip install "apache-airflow[postgres]==2.5.0" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.5.0/constraints-3.7.txt"
pip install pandas apache-airflow-providers-snowflake==2.1.0 snowflake-connector-python==2.5.1 snowflake-sqlalchemy==1.2.5

#Database Configuration
sudo apt-get install postgresql postgresql-contrib
sudo -i -u postgres
psql
CREATE DATABASE airflow;
CREATE USER airflow WITH PASSWORD 'airflow';
GRANT ALL PRIVILEGES ON DATABASE airflow TO airflow;
exit
exit

#Airflow Configuration

cd airflow
sed -i 's#sqlite:////home/ubuntu/airflow/airflow.db#postgresql+psycopg2://airflow:airflow@localhost/airflow#g' airflow.cfg
sed -i 's#SequentialExecutor#LocalExecutor#g' airflow.cfg
airflow db init
airflow users create -u airflow -f airflow -l airflow -r Admin -e airflow1@gmail.com

##Snowflake Queries:
------------------------
drop database if exists s3_to_snowflake;

use role accountadmin;
--Database Creation 
create database if not exists s3_to_snowflake;

--Specify the active/current database for the session.
use s3_to_snowflake;




create or replace stage s3_to_snowflake.PUBLIC.snow_simple url="s3://irisseta/output_folder/" 
credentials=(aws_key_id=''
aws_secret_key='');



list @s3_to_snowflake.PUBLIC.snow_simple;

--File Format Creation
create or replace file format my_parquet_format
type = parquet;



--Table Creation
create or replace external table s3_to_snowflake.PUBLIC.Iris_dataset  (CLASS_NAME varchar(20) as (Value:CLASS_NAME::varchar),
Count_Value Number as (Value:count::Number)) with location = @s3_to_snowflake.PUBLIC.snow_simple
file_format ='my_parquet_format';


select * from s3_to_snowflake.PUBLIC.Iris_dataset;



Create Snowflake Connection
