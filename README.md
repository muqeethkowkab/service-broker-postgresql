# Cloud Foundry Service Broker for a PostgreSQL instance

The broker currently publishes a single service and plan for provisioning PostgreSQL databases as database in a shared-vm.

This is using PostgreSQL jdbc driver 42 for PostgreSQL 9.

`This is a fork ! March 2017`
The project was forked on March 2017 (see github) because upstream where using a deprecated CC API not working with PCF 1.9+ or had serious issues in service binding (https://github.com/avasseur-pivotal/postgresql-cf-service-broker/issues/7).

## Design

The broker uses a PostgreSQL table for it's meta data. It does not maintain an internal database so it has no dependencies besides PostgreSQL.

Capability with the Cloud Foundry service broker API is indicated by the project version number. For example, version 2.8.0 is based off the 2.8 version of the broker API.

## Running

Simply run the JAR file and provide a PostgreSQL jdbc url via the `MASTER_JDBC_URL` environment variable.

It should look like expected by PostgreSQL driver:
```
jdbc:postgresql://localhost:5432/travis_ci_test?user=postgres&password=secret
```
(see `application.properties` or `manifest.yml` for override)

### Locally

```
mvn package && MASTER_JDBC_URL=jdbcurl java -jar target/postgresql-cf-service-broker-2.8.0-SNAPSHOT.jar
```

### In Cloud Foundry

Find out the database subnet and create a security group rule (postgresql.json):
```
[{"protocol":"tcp","destination":"10.10.8.0/24","ports":"5432"}]
```

import this into CF with:
```
cf create-security-group postgresql-service postgresql.json
```

Bind to the full cf install:
```
cf bind-running-security-group postgresql-service
```


Build the package with `mvn package` then push it out:
```
cf push postgresql-cf-service-broker -p target/postgresql-cf-service-broker-2.8.0-SNAPSHOT.jar --no-start
```

Export the following environment variables or better use a `manifest.yml`

```
...
env:
  SPRING_PROFILES_ACTIVE: cloud
  security.user.password: password
  MASTER_JDBC_URL: jdbc:postgresql://10.0.18.14:5432/sandbox?user=pgadmin&password=password
applications:
- name: ksb-postgresql
  path: target/postgresql-cf-service-broker-3.0.0-SNAPSHOT.jar
```

```
cf set-env postgresql-cf-service-broker MASTER_JDBC_URL "jdbc:postgresql://10.0.18.14:5432/sandbox?user=pgadmin&password=password"

cf set-env security.user.password "secret"
#or if you want:
cf set-env postgresql-cf-service-broker JAVA_OPTS "-Dsecurity.user.password=mysecret"
```

Start the service broker:
```
cf start postgresql-cf-service-broker
```

Create Cloud Foundry service broker:
```
cf create-service-broker postgresql-cf-service-broker user mysecret http://postgresql-cf-service-broker.bosh-lite.com
```

Add service broker to Cloud Foundry Marketplace for some or all organizations:
```
cf enable-service-access ksb-postgresql -p "shared-vm" -o ORG
```

## Testing

You need to have a running PostgreSQL 9.x instance for this to work locally.
To create an PostgreSQL database matching the ```MASTER_JDBC_URL```. Create the file  ```src/test/resources/application.properties``` and add as below (remember to replace the database name, username and password with the one you use):
```
security.user.password: password
service_id: pg
plan_id: postgresql-basic-plan
MASTER_JDBC_URL: jdbc:postgresql://localhost:5432/db?user=dbuser&password=password
```

Then run:
```
mvn test
```


## Broker Security

[spring-boot-starter-security](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-security) is used. See the documentation here for configuration: [Spring boot security](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-security)

The default password configured is "password"

## Creation of PostgreSQL databases

A service provisioning call will create a PostgreSQL database. A binding call will return a database uri that can be used to connect to the database. Unbinding calls will disable the database user role and deprovisioning calls will delete all resources created.

## User for Broker

An PostgreSQL user must be created for the broker. The username and password must be provided using the environment variable `MASTER_JDBC_URL`.

## Registering a Broker with the Cloud Controller

See [Managing Service Brokers](http://docs.cloudfoundry.org/services/managing-service-brokers.html).
