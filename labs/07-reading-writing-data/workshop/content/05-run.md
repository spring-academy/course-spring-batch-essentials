Now that second step is defined, let's add it to the sequence of steps in the job flow.

1. Add the new step to the job.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and update the bean definition of the `BillingJob` as follows:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public Job job(JobRepository jobRepository, Step step1, Step step2) {
   	return new JobBuilder("BillingJob", jobRepository)
   			.start(step1)
   			.next(step2)
   			.build();
   }
   ```

   Compared to the previous version, we added `step2` to the sequential flow using the `.next(step2)` method call. With that, Spring Batch will run the `fileIngestion` step after the `filePreparation` step. Let's run the job and check that.

1. Run the job.

   In the **Terminal** tab, run the following command to build the project:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Then, launch the job and pass the input file as a parameter with the following command:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2023-01.csv
   ```

   You should see something like this:

   ```shell
   2023-07-12T12:20:51.335+02:00  INFO 98965 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=BillingJob]] launched with the following parameters: [{'input.file':'{value=src/main/resources/billing-2023-01.csv, type=class java.lang.String, identifying=true}'}]
   2023-07-12T12:20:51.364+02:00  INFO 98965 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [filePreparation]
   2023-07-12T12:20:51.387+02:00  INFO 98965 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [filePreparation] executed in 22ms
   2023-07-12T12:20:51.407+02:00  INFO 98965 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [fileIngestion]
   2023-07-12T12:20:51.620+02:00  INFO 98965 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [fileIngestion] executed in 212ms
   2023-07-12T12:20:51.634+02:00  INFO 98965 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{'input.file':'{value=src/main/resources/billing-2023-01.csv, type=class java.lang.String, identifying=true}'}] and the following status: [COMPLETED] in 285ms
   ```

   This means the job was completed successfully. But, it's always good to verify the results are as expected.

1. Verify the job succeeded.

   Let's check the content of the `BILLING_DATA` table. In the **Terminal**, run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select count(*) from BILLING_DATA;'
   ```

   You should see that the table contains 1000 records, which is the number of lines in the input file.

It worked!
