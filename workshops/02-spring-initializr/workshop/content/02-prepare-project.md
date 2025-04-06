In this section, we prepare the project to use Spring Batch.

#### Java configuration

For a properly structured Spring Batch project, we should not put any batch configuration in the application's main class. Spring Batch's configuration should be defined in a separate class.

Open the editor in the `billing-job/src/main/java/example/billingjob` folder and create a new file named `BillingJobConfiguration.java` with the following content:

```editor:append-lines-to-file
file: ~/exercises/billing-job/src/main/java/example/billingjob/BillingJobConfiguration.java
description: "Create BillingJobConfiguration.java"
```

```java
package example.billingjob;

import org.springframework.context.annotation.Configuration;

@Configuration
public class BillingJobConfiguration {
  // TODO add job definition here
}
```

This class is a placeholder for Spring Batch related beans (Jobs, Steps, etc) and contains a `TODO` that you will implement in a future Lab.

#### Database configuration

1. Prepare the database with Spring Batch's metadata tables

   In this section, you will create Spring Batch's metadata tables in the database. This is the only time you will have to do that for learning purposes.
   Those tables will be pre-loaded in the database in all subsequent Labs of the course.

   Open the **Terminal** tab and run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker ps
   ```

   You should see a number of information about the Docker container running PostgreSQL. The output should be similar to the following:

   ```shell
   [~/exercises] $ docker ps
   CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                      NAMES
   c711e7873371   postgres:14.1-alpine   "docker-entrypoint.s…"   20 minutes ago   Up 20 minutes   127.0.0.1:5432->5432/tcp   postgres
   [~/exercises] $
   ```

   We are now going to connect to the database and create the tables. For that, use the following command:

   ```shell
   [~/exercises] $ docker exec -it postgres psql -U postgres
   psql (14.1)
   Type "help" for help.
   postgres=#
   ```

   You are now connected to the database with the `psql` command line tool. This tool is the official command line interface for PostgreSQL.
   It is now time to create the metadata tables.

   Copy the following SQL script, paste it in the **Terminal** and hit `enter`:

   ```shell
   CREATE TABLE BATCH_JOB_INSTANCE  (
       JOB_INSTANCE_ID BIGINT  NOT NULL PRIMARY KEY ,
       VERSION BIGINT ,
       JOB_NAME VARCHAR(100) NOT NULL,
       JOB_KEY VARCHAR(32) NOT NULL,
       constraint JOB_INST_UN unique (JOB_NAME, JOB_KEY)
   ) ;
   CREATE TABLE BATCH_JOB_EXECUTION  (
       JOB_EXECUTION_ID BIGINT  NOT NULL PRIMARY KEY ,
       VERSION BIGINT  ,
       JOB_INSTANCE_ID BIGINT NOT NULL,
       CREATE_TIME TIMESTAMP NOT NULL,
       START_TIME TIMESTAMP DEFAULT NULL ,
       END_TIME TIMESTAMP DEFAULT NULL ,
       STATUS VARCHAR(10) ,
       EXIT_CODE VARCHAR(2500) ,
       EXIT_MESSAGE VARCHAR(2500) ,
       LAST_UPDATED TIMESTAMP,
       constraint JOB_INST_EXEC_FK foreign key (JOB_INSTANCE_ID)
       references BATCH_JOB_INSTANCE(JOB_INSTANCE_ID)
   ) ;
   CREATE TABLE BATCH_JOB_EXECUTION_PARAMS  (
       JOB_EXECUTION_ID BIGINT NOT NULL ,
       PARAMETER_NAME VARCHAR(100) NOT NULL ,
       PARAMETER_TYPE VARCHAR(100) NOT NULL ,
       PARAMETER_VALUE VARCHAR(2500) ,
       IDENTIFYING CHAR(1) NOT NULL ,
       constraint JOB_EXEC_PARAMS_FK foreign key (JOB_EXECUTION_ID)
       references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
   ) ;
   CREATE TABLE BATCH_STEP_EXECUTION  (
       STEP_EXECUTION_ID BIGINT  NOT NULL PRIMARY KEY ,
       VERSION BIGINT NOT NULL,
       STEP_NAME VARCHAR(100) NOT NULL,
       JOB_EXECUTION_ID BIGINT NOT NULL,
       CREATE_TIME TIMESTAMP NOT NULL,
       START_TIME TIMESTAMP DEFAULT NULL ,
       END_TIME TIMESTAMP DEFAULT NULL ,
       STATUS VARCHAR(10) ,
       COMMIT_COUNT BIGINT ,
       READ_COUNT BIGINT ,
       FILTER_COUNT BIGINT ,
       WRITE_COUNT BIGINT ,
       READ_SKIP_COUNT BIGINT ,
       WRITE_SKIP_COUNT BIGINT ,
       PROCESS_SKIP_COUNT BIGINT ,
       ROLLBACK_COUNT BIGINT ,
       EXIT_CODE VARCHAR(2500) ,
       EXIT_MESSAGE VARCHAR(2500) ,
       LAST_UPDATED TIMESTAMP,
       constraint JOB_EXEC_STEP_FK foreign key (JOB_EXECUTION_ID)
       references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
   ) ;
   CREATE TABLE BATCH_STEP_EXECUTION_CONTEXT  (
       STEP_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
       SHORT_CONTEXT VARCHAR(2500) NOT NULL,
       SERIALIZED_CONTEXT TEXT ,
       constraint STEP_EXEC_CTX_FK foreign key (STEP_EXECUTION_ID)
       references BATCH_STEP_EXECUTION(STEP_EXECUTION_ID)
   ) ;
   CREATE TABLE BATCH_JOB_EXECUTION_CONTEXT  (
       JOB_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
       SHORT_CONTEXT VARCHAR(2500) NOT NULL,
       SERIALIZED_CONTEXT TEXT ,
       constraint JOB_EXEC_CTX_FK foreign key (JOB_EXECUTION_ID)
       references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
   ) ;
   CREATE SEQUENCE BATCH_STEP_EXECUTION_SEQ MAXVALUE 9223372036854775807 NO CYCLE;
   CREATE SEQUENCE BATCH_JOB_EXECUTION_SEQ MAXVALUE 9223372036854775807 NO CYCLE;
   CREATE SEQUENCE BATCH_JOB_SEQ MAXVALUE 9223372036854775807 NO CYCLE;
   ```

   Finally, check that all tables were correctly created using the `\d` command:

   ```shell
   postgres=# \d
                         List of relations
    Schema |             Name             |   Type   |  Owner
   --------+------------------------------+----------+----------
    public | batch_job_execution          | table    | postgres
    public | batch_job_execution_context  | table    | postgres
    public | batch_job_execution_params   | table    | postgres
    public | batch_job_execution_seq      | sequence | postgres
    public | batch_job_instance           | table    | postgres
    public | batch_job_seq                | sequence | postgres
    public | batch_step_execution         | table    | postgres
    public | batch_step_execution_context | table    | postgres
    public | batch_step_execution_seq     | sequence | postgres
   (9 rows)
   postgres=#
   ```

2. Configure database properties in Spring Boot

   Open the `billing-job/src/main/resources/application.properties` and add the following properties:

   ```editor:open-file
   file: ~/exercises/billing-job/src/main/resources/application.properties
   description: "Open application.properties"
   ```

   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
   spring.datasource.username=postgres
   spring.datasource.password=postgres
   ```

   With those properties in place, Spring Boot will auto-configure a JDBC `DataSource` bean in the application context
   that we can use in our batch application.
