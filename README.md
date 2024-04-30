# Building a simple End-to-End Data Engineering System 
This project uses different tools such as kafka, airflow, spark, postgres and docker. 

Reference to this pipeline: https://medium.com/@hamzagharbi_19502/end-to-end-data-engineering-system-on-real-data-with-kafka-spark-airflow-postgres-and-docker-a70e18df4090

## Overview

1. Data Streaming: Initially, data is streamed from the API into a Kafka topic.
  
2. Data Processing: A Spark job then takes over, consuming the data from the Kafka topic and transferring it to a PostgreSQL database.
   
3. Scheduling with Airflow: Both the streaming task and the Spark job are orchestrated using Airflow. While in a real-world scenario, the Kafka producer would constantly listen to the API, for demonstration purposes, we'll schedule the Kafka streaming task to run daily. Once the streaming is complete, the Spark job processes the data, making it ready for use by the LLM application.

All of these tools will be built and run using docker, and more specifically docker-compose.

![chatuml-diagram](https://github.com/HamzaG737/data-engineering-project/assets/71135893/ce92b731-038a-4d9c-9722-f97a6ba51153)

## Prerequisites
1. A ubuntu 22.04 VM with 4 cpu 16 GB RAM
2. Install Docker Version: 26.1.0 or above
   
    You may follow this link for intallation:
    https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04
4. Install Python:
   
    sudo apt-get update
   
    sudo apt install python3-pip
6. Install docker compose
   
    sudo apt  install docker-compose
8. Install postgres
   
   a. You may follow this link to install the postgres DB
     https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-22-04-quickstart

   b. Once postgres installed the set the password for 'postgres' user, using the below commands:

   sudo -i -u postgres

   psql

   ALTER USER postgres WITH PASSWORD 'welcome';

   c. once passwprd is set, then test the login

   sudo -i -u postgres
   
   psql -h localhost postgres

   d. It will prompt for password enter the newly created password as 'welcome'
   
   e. Now to connect postgres DB remotrely, we need to do some setup

      1. open /etc/postgresql/14/main/postgresql.conf, set the "listen_addresses" to "*"

      listen_addresses = '*'          # what IP address(es) to listen on;

      2. open sudo vi /etc/postgresql/14/main/pg_hba.conf, add the following line:
   
      host    all             all             0.0.0.0/0               md5

      3. restart the postgres db as:

      sudo service postgresql restart : to start the server

      4. check the status of the DB:

      sudo service postgresql status : to check server status   
  
## Steps to run this project
1. git clone https://github.com/debkantap/data_pipeline.git
2. got to 'data_pipeline' directory
3. Install the requirements:
    pip install -r requirements.txt
4. create the docker network
    sudo docker network create airflow-kafka
    sudo docker network ls
5. export POSTGRES_PASSWORD=welcome
6. echo $POSTGRES_PASSWORD
7. python3 scripts/create_table.py
8. open the src/constants.py
   
   modify the file, replace "POSTGRES_URL = f"jdbc:postgresql://<ip address>:5432/postgres", "ip address" with the ip address postgres. Also "password": "welcome",
9. Set the postgres password in Dockerfile as well. spark/Dockerfile
10. Set the date in ./data/last_processed.json to long previous date
11. create the spark job image

sudo docker build -f spark/Dockerfile -t rappel-conso/spark:latest --build-arg POSTGRES_PASSWORD=welcome  .

12. sudo docker-compose up -d
    
14. echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_PROJ_DIR=\"./airflow_resources\"" > .env

16. sudo docker compose -f docker-compose-airflow.yaml up -d
    
18. The airflow UI will be avilable in: http://<IP of the VM>:8080/
    
20. Login to portal as airflow/airflow
    
22. Scroll down to find "kafka_spark_dag" workflow

24. Run worflow to see the result. Data must be populated in "rappel_conso_table" table

    
   

