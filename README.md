# My temporal deployment

This repository contains, step by step, how to deploy [temporal](https://temporal.io/) form scratch. 

## Configuration

### Create postgres DB

```
docker run \
    --name temporal_postgres \
    -p 5432:5432 \
    -e POSTGRES_USER=temporal \
    -e POSTGRES_PASSWORD=temporal \
    --rm \
    -v ./db-data:/var/lib/postgresql/data <!--to persist data in db-data -->
    postgres:13 
```

> https://github.com/temporalio/temporal/tree/master/tools/sql



#### DB configuration

before...

Download and unzip temporal server

https://github.com/temporalio/temporal/releases/tag/v1.21.2


```
curl -o temporal_tmp https://github.com/temporalio/temporal/releases/download/v1.21.2/temporal_1.21.2_darwin_arm64.tar.gz
unzip temporal_tmp -d temporal
rm -f temporal_tmp


curl https://github.com/temporalio/temporal/releases/download/v1.21.2/temporal_1.21.2_darwin_arm64.tar.gz -L -o temporal.tar.gz && \
tar -xvf temporal.tar.gz && \
rm -f temporal.tar.gz

```




#### Create persistence store

```
export SQL_HOST=localhost
export port=5432
./temporal-sql-tool --plugin postgres --ep  $SQL_HOST -p $port -u temporal -pw temporal --db temporal create
./temporal-sql-tool --plugin postgres --ep $SQL_HOST -p $port -u temporal -pw temporal --db temporal setup-schema -v 0.0
./temporal-sql-tool --plugin postgres --ep $SQL_HOST -p $port -u temporal -pw temporal --db temporal update-schema -d ./../schema/postgresql/v96/temporal/versioned 
```


#### Create visibility store

```
export SQL_HOST=localhost
export port=5432

./temporal-sql-tool --plugin postgres --ep $SQL_HOST -p $port -u temporal -pw temporal --db temporal_visibility create
./temporal-sql-tool --plugin postgres --ep $SQL_HOST -p $port -u temporal -pw temporal --db temporal_visibility setup-schema -v 0.0
./temporal-sql-tool --plugin postgres --ep $SQL_HOST -p $port -u temporal -pw temporal --db temporal_visibility update-schema -d ./../schema/postgresql/v96/visibility/versioned 

```


### Start temporal server


Since we are using postgres as a visibility store, we have to export the following environment variables:

```
export DB=postgresql
export DB_PORT=5432
export POSTGRES_USER=postgres
export POSTGRES_PWD=temporal
export POSTGRES_SEEDS=postgresql
export TEMPORAL_ENVIRONMENT=development-postgres
export TEMPORAL_CONFIG_DIR=./config
```


<!--
// export TEMPORAL_DYNAMIC_CONFIG_FILE_PATH=..config/development.yaml
-->



There are several ways to start temporal, you can use either a docker image or run the 
binary directly, I am going to opt for the latest one. 



Now let's start temporal server. 

`./temporal-server start`

<!--


Alternatively you can start temporal using the docker image

```
docker run \
    -e LOG_LEVEL=debug,info \
    -e DYNAMIC_CONFIG_FILE_PATH=config/docker.yaml \
    temporalio/server:1.21.1
//ADD env variables
```
-->


And the temporal UI. For the sake of simplicity we are using the docker image. 

```
docker run \
    --name temporal_ui \
    -p 8081:8081 \
    -e TEMPORAL_ADDRESS=host.docker.internal:7233 \
    -e TEMPORAL_UI_PORT=8081 \
    --add-host host.docker.internal:host-gateway \
    --rm \
    temporalio/ui:2.16.1
```
Navigate to http://localhost:8081


There is no namespaces created, you can create the first namespace using the 
following command

`temporal operator namespace create default --description default`

