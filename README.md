# My temporal deployment

This repository contains how to deploy temporal.io form scratch. 

Temporal is an orchestration engine that will allow you to build reliable applications. 


## Configuration

### Create postgres DB

```
docker run \
    --name temporal_postgres \
    -p 5432:5432 \
    -e POSTGRES_USER=temporal \
    -e POSTGRES_PASSWORD=temporal \
    --rm \
    postgres:13 
```

> https://github.com/temporalio/temporal/tree/master/tools/sql

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

