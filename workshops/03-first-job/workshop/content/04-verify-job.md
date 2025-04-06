Let's check the `Job` status in the database.

1. Verify the `Job` status.

   In the **Terminal** tab, run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION;'
   ```

   You should see something like the following output:

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION;'
   job_execution_id | version | job_instance_id |        create_time         | start_time | end_time |  status  | exit_code | exit_message |        last_updated
   ------------------+---------+-----------------+----------------------------+------------+----------+----------+-----------+--------------+----------------------------
                   1 |       0 |               1 | 2023-04-28 08:28:52.033414 |            |          | STARTING | UNKNOWN   |              | 2023-04-28 08:28:52.033859
   (1 row)

   [~/exercises] $
   ```

   Spring Batch has correctly recorded the execution of the `Job` in the database, but the `Job`'s status is `STARTING` and its exit code is `UNKNOWN`.
   How is that possible when our `Job` was successfully executed and completed?

   This is where the responsibility of the `Job` to report its status and exit code to the `JobRepository` comes to play. Let's fix that.

2. Set statuses and utilize the `JobRepository`.

   Open the `src/main/java/example/billingjob/BillingJob.java` file and update its content as follows:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJob.java
   description: "Open BillingJob.java"
   ```

   ```java
   package example.billingjob;

   import org.springframework.batch.core.BatchStatus;
   import org.springframework.batch.core.ExitStatus;
   import org.springframework.batch.core.Job;
   import org.springframework.batch.core.JobExecution;
   import org.springframework.batch.core.repository.JobRepository;

   public class BillingJob implements Job {

       private JobRepository jobRepository;

       public BillingJob(JobRepository jobRepository) {
           this.jobRepository = jobRepository;
       }

       @Override
       public String getName() {
           return "BillingJob";
       }

       @Override
       public void execute(JobExecution execution) {
           System.out.println("processing billing information");
           execution.setStatus(BatchStatus.COMPLETED);
           execution.setExitStatus(ExitStatus.COMPLETED);
           this.jobRepository.update(execution);
       }
   }
   ```

   In this step, we first passed a `JobRepository` reference as a constructor parameter to our `BillingJob`. We will use the `JobRepository` to save important information about our `Job`.

   ```java
   private JobRepository jobRepository;

   public BillingJob(JobRepository jobRepository) {
       this.jobRepository = jobRepository;
   }
   ```

   Next, we have updated the `execute` method to set the `Job`'s execution status as well as its exit status:

   ```java
   @Override
   public void execute(JobExecution execution) {
       System.out.println("processing billing information");
       execution.setStatus(BatchStatus.COMPLETED);
       execution.setExitStatus(ExitStatus.COMPLETED);
       this.jobRepository.update(execution);
   }
   ```

   Finally, we issued a `jobRepository.update(execution)` to update the `Job` execution in the database.

3. Understand the updates.

   What you need to understand here is that it is the responsibility of the `Job` implementation to report its status to the `JobRepository`.

   - The batch `Job` status indicates the status of the execution.

     For example, if the `Job` is running the batch status is `BatchStatus.STARTED`. If it fails, it's `BatchStatus.FAILED`, and if it finishes successfully, it's `BatchStatus.COMPLETED`.

   - Since our `Job` was completed successfully we will set the execution status to `BatchStatus.COMPLETED` and the exit status to `ExitStatus.COMPLETED` as well.

   Now that we updated our `Job` implementation to use a `JobRepository` to report its status, we need to add a reference to the `JobRepository` in our `Job` bean definition.

4. Supply the `JobRepository` to the `Job`.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and update its content as follows:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   package example.billingjob;

   import org.springframework.batch.core.Job;
   import org.springframework.batch.core.repository.JobRepository;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;

   @Configuration
   public class BillingJobConfiguration {

       @Bean
       public Job job(JobRepository jobRepository) {
           return new BillingJob(jobRepository);
       }
   }
   ```

   Thanks to Spring Boot, a `JobRepository` was autoconfigured with the datasource configured for our PostgreSQL database.

   This `JobRepository` is ready for us to use by autowiring it in our `Job` bean.

5. Clean up and re-run the `Job`.

   Now let's try to re-run the `Job` and check its status in the database.

   But before re-running the `Job`, let's clean the database up to cut the noise of the previous run. In the **Terminal** tab, run the following commands:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ scripts/drop-create-database.sh
   ```

   Now re-run the `Job` as shown before and check the database. The execution status as well as the exit status should now be `COMPLETED`.

   ```shell
   ...
   2023-05-03T21:21:07.939Z  INFO 7458 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [example.billingjob.BillingJob@66f0548d] completed with the following parameters: [{}] and the following status: [COMPLETED]
   ```

   If the `Job`'s status is `COMPLETED`, then congratulations! You successfully created, configured and run your first Spring Batch `Job`!

Now the question is: what happens if an error occurs while performing the business logic?

Let's learn about that next.
