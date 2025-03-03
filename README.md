# Data Portal

![badge1](https://img.shields.io/badge/language-Python-blue.svg)
![badge2](https://img.shields.io/badge/orchestrator-Airflow-brightgreen.svg)
![badge3](https://img.shields.io/badge/containerization-Docker-red.svg)

## 📑 General

This repository is associated to the data portal of the University of Luxembourg managed by the [MobiLab](https://mobilab.lu/) research group.


## 📂 Framework

The data portal is based on the following framework:

![Diagram showing the framework of the data portal](readme-resources/framework.png)

- **Data sources**:
The data sources are the different data providers that are used to feed the data portal. 
    - `Bike data`: The bike data is provided by the [jcdecaux](https://api.jcdecaux.com/vls/v1/stations?contract=Luxembourg&apiKey=4507a17cda9135dd36b8ff13d8a4102ab3aa44a0) service.
    - `Charging stations data`: The charging stations data is provided by the [data public lu](https://data.public.lu) service.
    - `Traffic counters data`: The traffic counters data is provided by the [cita lu](http://www.cita.lu) service.
    - `Stops public transport data`: The stops public transport data is provided by the [mobiliteit lu](https://data.public.lu/en/datasets/api-mobiliteit-lu/) service.
    - `Departure board data`: The departure board data is provided by the [mobiliteit lu](https://data.public.lu/en/datasets/api-mobiliteit-lu/) service.
    - `Parking data`: The parking data is provided by the [Ville de Luxembourg](https://www.vdl.lu) website.
    - `Google Popular Times data`: The Google popular Times data is provided by the [Google Maps](https://www.google.com/maps) service.
<br/>
     
- **Data Extraction**:
The data extraction is performed using Python as main programming language, using libraries such as `requests`, and `selenium` with `chrome driver` and `chrome browser`. 
<br/>

- **Data Transformation**:
The data transformation is performed using Python as main programming language, using libraries such as `pandas`.
<br/>

- **Data Loading**:
Finally, the data is loaded into a [PostgreSQL](https://www.postgresql.org/) database.
<br/>

- **DB Management**:
[Ngrok](https://ngrok.com/) is used to expose the PostgreSQL database to the internet.
Then, [DBeaver](https://dbeaver.io/) is used to manage the database, even though it can be managed using other tools such as [DataGrip](https://www.jetbrains.com/datagrip/).
<br/>

The entire pipeline is containerized using [Docker](https://www.docker.com/), and it is orchestrated using [Apache Airflow](https://airflow.apache.org/).
Finally, all the resources are stored in a [GitHub](https://github.com/jdpinedaj/luxmobi) repository.


## 📚 Workflows
There are four workflows in the data portal:

- **workflow_create_tables**: This workflow creates the tables in the database. This is the first workflow that should be executed, and it is executed only once.

- **workflow_ETL**: This workflow performs the ETL process for each data source. It is executed every day at 00:00, and it is necessary to mention that the start_date of the DAG has to be set to the current date for the first time it is executed, as mentioned in the file itself.

- **workflow_load_csv_data**: In case the airflow server is down for some reason, this workflow can be executed to load the data from the CSV files that were already created by the previous workflow. It is important to note that this workflow should be executed only once, as a backup solution, in case the airflow server is down for some reason and it is needed to clone the repository and mount the containers again.

- **workflow_integration_db**: This workflow performs the integration of the data from the previous MySQL database to the current PostgreSQL database. It is important to note that this workflow should be executed only once, as a backup solution. And still has to be tested.

TBD: change the structure of PG db such that they can be merged with our DB. 


---

## 💾 Running Airflow on Docker

#### 1. docker-compose.yaml

To deploy Airflow on Docker Compose, you should fetch docker-compose.yaml.

```
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.6.0/docker-compose.yaml'
```

In addition, please include the following element in the postgres service:

```
    ports:
        - "5432:5432"
```

#### 2. Initializing environment

On Linux, the quick-start needs to know your host user id and needs to have group id set to 0. Otherwise the files created in dags, logs and plugins will be created with root user. You have to make sure to configure them for the docker-compose:

```
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

#### 3. Initialize the database

First, start docker desktop. 

- On Windows, you just need to start Docker Desktop.


- On Linux, you need to enable docker-desktop service:
    
```
systemctl --user enable docker-desktop
```

Then, on all operating systems, you need to run database migrations and create the first user account. To do it, run.

```
docker-compose up -d airflow-init
```

#### 4. Cleaning-up the environment

The docker-compose we prepare is a “Quick-start” one. It is not intended to be used in production and it has a number of caveats - one of them being that the best way to recover from any problem is to clean it up and restart from the scratch.

The best way to do it is to:

- Run docker-compose down --volumes --remove-orphans command in the directory you downloaded the docker-compose.yaml file
- remove the whole directory where you downloaded the docker-compose.yaml file rm -rf '<DIRECTORY>'
- re-download the docker-compose.yaml file
- re-start following the instructions from the very beginning in this guide

#### 5. Running Airflow

Now you can start all services:

```
docker-compose up -d
```

#### 6. Accessing the environment via a browser using the web interface

Check http://localhost:8081
Or http://host.docker.internal:8081

login credentials:
usr: airflow
psw: airflow

#### 7. Extending the image to include custom packages after initialization of the database

If you want to install python requirements OR when you open the localhost you see an error like 'DAG Import errors', this is needed to be done. You need to run the python requirements to the airflow container after the initialization of the database. This can be done by:

- adding the requirements to the *requirements.txt* file
- rebuilding the image docker-compose build by running `docker-compose build --no-cache`, after commenting and uncommenting the respective lines at the beginning of the *docker-compose.yaml* file. (go onto docker compose, comment the line image etc.. (line 53) and decomment line build (line 54))
- restarting the containers by running `docker-compose up -d`

#### FIRST RUN #####
Most of these commands below are for running it for the first time, or when it breaks.
#### 8. Airflow connection and Postgres Operator

In Airflow/Admin/Connections
- go on + -> add a new record
- Connection ID: `postgres_default`
- Connection type: `postgres`
- Host: `postgres` 
- Schema: `luxmobi`
- Login: `nipi`
- Password: `MobiLab1`
- Port: `5432`

#### 8.2. workflow_ETL.py change

put create_tables in green, check if it is working (it should work properly).

#### 8.1. workflow_ETL.py change
for the first time you're running the service, PLEASE ensure that you change the start date on the actual date you're doing this. (Line 45 of workflow_ETL.py, UTC date)

TBD: check on workflow_ETL.py a way to avoid this. Remember that this code runs is checking on the file, so the start date should be on the install date, BUT it should stay the install date without changing at every call.

Then go to airflow again, open workflow_ETL and run 

#### 9. If there are some problems starting postgres:

```
docker exec -it luxmobi-postgres-1 psql -U nipi -d luxmobi -c "CREATE ROLE airflow LOGIN PASSWORD 'airflow';"
docker exec -it luxmobi-postgres-1 psql -U nipi -d luxmobi -c "CREATE DATABASE airflow;"
```

#### IF EVERYTHING GOES DOWNHILL SUPERFAST ####
- you don't see the workflow
- DAG is not in the bag
- errors that exist there


Only solution is removing everything and reinstalling everything again.

#### 10. Reseting Docker to start again

In case you need to reinstall everything again, you just need to run:

```
docker-compose down --volumes --rmi all
```

Or, if you want to prune everything: (maybe twice or 3 times...)

```
docker system prune -a
```

Then go to docker, remove EVERYTHING in container, images, volumes. 

Then you go to dags/data AND SAVE THE DATA YOU STORED.

Remove also luxmobi folder from the computer and restart the pc.

And then start again from 1.

Then, when you finish again, take back the data you SAVED BEFORE and copy them in dags/data. So the system will think that nothing happened.

If this happens, you need to trigger the emergency botton (read -> workflow_load_csv_data in airflow)

#### 11. In case the DAGS are not visible in the webserver

In case you can't see the DAGS in the UI, you can try to run the following command to check if the DAGS are there:
```
docker exec -it luxmobi-airflow-webserver-1 airflow dags list
```

Also, you can try to access the webserver container:
```
docker exec -it luxmobi-airflow-webserver-1 bash
```

Then, you have to navigate to the DAGS folder. First run the following to check the files in the current directory:
```
ls -l
```

Then, navigate to the folder dags:
```
cd dags
```
and then, again
```
ls -l
```

If you can see the DAGS, then it is better to clone the repository again and start again.

In case you follow this solution, it is necessary to manually copy-paste the CSV files from the previous run into the new repository, and then run the `workflow_load_csv_data` workflow after creating the tables, with the aim of loading the data from the CSV files that were already created by the previous workflow.


#### 12. Useful commands
    
- To see the logs of a specific container:
```
docker logs -f <container_name>
```

- To see the dag list
```
docker exec -it luxmobi-airflow-webserver-1 airflow dags list
```

#### 13. IN CASE EVER


#### More info

For more info, please visit:
https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html
