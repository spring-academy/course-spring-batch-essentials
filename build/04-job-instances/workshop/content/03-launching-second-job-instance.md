In this section, we launch a second `JobInstance` to process the billing data set of February 2023, which is in the `src/main/resources/billing-2023-02.csv` file.

1. Launch the `BillingJob` and pass the `billing-2023-02.csv` file as a `JobParameter`.

   In the **Terminal** tab, run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2023-02.csv
   ```

   You should see the following message in the console:

   ```shell
   processing billing information from file src/main/resources/billing-2023-02.csv
   ```

   This means our `BillingJob` has correctly processed the data of February 2023 and generated the report.

   Now let's check the database to inspect the second `JobInstance` details.

2. Inspect the Batch metadata in the database.

   ```dashboard:open-dashboard
   name: Terminal
   ```

   In the **Terminal** tab, run the following command:

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_INSTANCE;'
   ```

   You should see something like the following output:

   ```shell
   job_instance_id | version |  job_name  |             job_key
   ----------------+---------+------------+----------------------------------
                 1 |       0 | BillingJob | c0cb4257f9f2b2fa119bbebfb801772f
                 2 |       0 | BillingJob | f70523e06481f0c914d3bdb634b86802
   (2 rows)
   ```

   Now we have a second `JobInstance` with ID `2` for the same `BilingJob`. Note how the `job_key` is different due to the different identifying `JobParameter`.

   Now let's check the corresponding `JobExecution` for this second `JobInstance`. For that, use the following command:

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION;'
   ```

   You should see something like the following output:

   ```shell
   job_execution_id | version | job_instance_id |        create_time         | start_time | end_time |  status   | exit_code | exit_message |        last_updated
   -----------------+---------+-----------------+----------------------------+------------+----------+-----------+-----------+--------------+----------------------------
                  1 |       1 |               1 | 2023-05-25 10:05:45.629492 |            |          | COMPLETED | COMPLETED |              | 2023-05-25 10:05:45.642106
                  2 |       1 |               2 | 2023-05-26 07:24:18.730961 |            |          | COMPLETED | COMPLETED |              | 2023-05-26 07:24:18.742902
   (2 rows)
   ```

   As expected, a second `JobExecution` for this second `JobInstance` is now present in the table.

   Finally, let's check the content of the `BATCH_JOB_EXECUTION_PARAMS` with the following command:

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION_PARAMS;'
   ```

   You should see something like the following output:

   ```shell
   job_execution_id | parameter_name |  parameter_type  |            parameter_value             | identifying
   -----------------+----------------+------------------+----------------------------------------+-------------
                  1 | input.file     | java.lang.String | src/main/resources/billing-2023-01.csv | Y
                  2 | input.file     | java.lang.String | src/main/resources/billing-2023-02.csv | Y
   (2 rows)
   ```

   As you can see, each execution has its own parameter recorded in the `BATCH_JOB_EXECUTION_PARAMS` table.
