In the previous lesson, we discussed `JobInstance`s and showed how they are defined and identified by `JobParameter`s.
In this Lab, you will learn how to pass parameters to a `Job` in order to create `JobInstance`s.

Our Spring Cellular `BillingJob` is expected to process monthly billing data from a flat file.
The input file will be used as an identifying `Job` parameter named `input.file`. Therefore, we would have a distinct `JobInstance` per month.

The `src/main/resources` folder contains two flat files, `billing-2023-01.csv` and `billing-2023-02.csv` containing respectively the billing data for January 2023 and February 2023.
You can explore the data in those files, but the content is not relevant to this Lab for the moment. We will explain the format of these files and what data they represent in a future Lab.
For now, all you have to understand is that we will pass these files as parameters to our `BillingJob` to create distinct `JobInstance`s.

1. Modify the `BillingJob` implementation to get the input file from `JobParameter`s.

   In the `Editor` tab, go to the `src/main/java/example/billingjob/BillingJob.java` and update the `execute` method with the following code:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJob.java
   text: "public void execute"
   description: "Open BillingJob.java"
   ```

   ```java
    @Override
    public void execute(JobExecution execution) {
        JobParameters jobParameters = execution.getJobParameters();
        String inputFile = jobParameters.getString("input.file");
        System.out.println("processing billing information from file " + inputFile);
        execution.setStatus(BatchStatus.COMPLETED);
        execution.setExitStatus(ExitStatus.COMPLETED);
        this.jobRepository.update(execution);
    }
   ```

   You should also add the following import statement:

   ```java
   import org.springframework.batch.core.JobParameters;
   ```

   In this section, we get access to `JobParameter`s from the `JobExecution` reference and extract the `input.file` parameter. We then updated the message we print to the standard output to show which file the current execution is processing.

2. Launch the `BillingJob` and pass the the billing data input file as a `JobParameter`.

   In the **Terminal** tab, run the following command to build the project:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw package -Dmaven.test.skip=true
   ```

   Then, launch the `Job` and pass the input file as a parameter with the following command:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2023-01.csv
   ```

   You should see the following message in the console:

   ```shell
   processing billing information from file src/main/resources/billing-2023-01.csv
   ```

   Great! This means our `Job` is now able to get the input file from `JobParameter`s and process the data as expected. Now let's check the database to inspect the first `JobInstance` details.

3. Inspect the Batch metadata in the database.

   In the **Terminal** tab, run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_INSTANCE;'
   ```

   You should see something like the following output:

   ```shell
   job_instance_id | version |  job_name  |             job_key
   ----------------+---------+------------+----------------------------------
                 1 |       0 | BillingJob | c0cb4257f9f2b2fa119bbebfb801772f
   (1 row)
   ```

   We have a first `JobInstance` with ID `1` for the `Job` named `BilingJob`.
   The `version` column is a technical column used by Spring Batch for optimistic locking, and its usage is out of scope of this Lab.
   The `job_key` is a hash of the identifying `JobParameter`s calculated by Spring Batch to identify `JobInstance`s.

   For more details about this matter, please refer to the external resources `Job instance identification` and `Job keys generation APIs` in the previous lesson.

   Now let's check the corresponding `JobExecution` for this first `JobInstance`. For that, use the following command:

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION;'
   ```

   You should see something like the following output:

   ```shell
   job_execution_id | version | job_instance_id |        create_time         | start_time | end_time |  status   | exit_code | exit_message |        last_updated
   -----------------+---------+-----------------+----------------------------+------------+----------+-----------+-----------+--------------+----------------------------
                  1 |       1 |               1 | 2023-05-25 10:05:45.629492 |            |          | COMPLETED | COMPLETED |              | 2023-05-25 10:05:45.642106
   (1 row)
   ```

   As we see, we have a first execution that corresponds to the first instance (through the `job_instance_id`) which is successfully completed (`status=COMPLETED`).

   Finally, let's check the content of the `BATCH_JOB_EXECUTION_PARAMS` with the following command:

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION_PARAMS;'
   ```

   You should see something like the following output:

   ```shell
   job_execution_id | parameter_name |  parameter_type  |            parameter_value             | identifying
   -----------------+----------------+------------------+----------------------------------------+-------------
                  1 | input.file     | java.lang.String | src/main/resources/billing-2023-01.csv | Y
   (1 row)
   ```

   As expected, the first execution received the `input.file` `Job` parameter with the value `src/main/resources/billing-2023-01.csv`.

This all works wonderfully! But what if we re-run the same `JobInstance` even if it completes successfully? Let's try it and find out.
