To understand better how Spring Batch keeps track of our job's successes and failures, we need to look at Spring Batch's metadata in the database.

Specifically, let's look at the data we have for job executions and step executions.

1. Inspect the metadata of job executions.

   Open a **Terminal** and run the following query to inspect the `BATCH_JOB_EXECUTION` table:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select job_instance_id, job_execution_id, status from BATCH_JOB_EXECUTION;'
   ```

   You should see something like the following:

   ```shell
    job_instance_id | job_execution_id |  status
   -----------------+------------------+-----------
                  1 |                1 | FAILED
                  1 |                2 | FAILED
                  1 |                3 | COMPLETED
   (3 rows)
   ```

   As you see, there are 3 job executions for the same job instance with `job_instance_id` of `1`. These correspond to the set of parameters that we passed on the command line.

   The first two job executions have failed, while the last one succeeded. This is the restartability feature of Spring Batch in action!

   Indeed, it was possible to fix the data and restart the job until it succeeded.

   Next, let's check the metadata of step executions.

1. Inspect the metadata of step executions.

   Open a **Terminal** and run the following query to inspect the `BATCH_STEP_EXECUTION` table:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select step_execution_id, job_execution_id, step_name, status, read_count, write_count, commit_count, rollback_count  from BATCH_STEP_EXECUTION;'
   ```

   You should see the following output:

   ```shell
    step_execution_id | job_execution_id |    step_name     |  status   | read_count | write_count | commit_count | rollback_count
   -------------------+------------------+------------------+-----------+------------+-------------+--------------+----------------
                    1 |                1 | filePreparation  | COMPLETED |          0 |           0 |            1 |              0
                    2 |                1 | fileIngestion    | FAILED    |        225 |         200 |            2 |              1
                    3 |                2 | fileIngestion    | FAILED    |        207 |         200 |            2 |              1
                    4 |                3 | fileIngestion    | COMPLETED |        100 |         100 |            2 |              0
                    5 |                3 | reportGeneration | COMPLETED |        500 |         388 |            6 |              0
   ```

   This output shows several keys points:

   - The `filePreparation` step has been run only once. In fact, it succeeded during the first job execution and Spring Batch did not run it again.
   - The `fileIngestion` step failed twice and succeeded the third time, which corresponds to the result we experienced during the restart attempts of our job.

     Note how read counts and write counts also correlate to the lines we fixed in the data file.

   - Finally, the `reportGeneration` step was executed only once at the third and final attempt, once all the data was fixed and correctly ingested into the `BILLING_DATA` table.

This data can be very valuable, especially if you need to debug how and where job failures are occurring in your jobs.
