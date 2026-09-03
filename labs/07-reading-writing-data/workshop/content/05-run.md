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
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2026-01.csv
   ```

   You should see something like this:

   ```shell
   ...
   2026-08-21T12:49:49.316Z  INFO 4539 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=BillingJob]] launched with the following parameters: [{JobParameter{name='input.file', value=src/main/resources/billing-2026-01.csv, type=class java.lang.String, identifying=true}}]
   2026-08-21T12:49:49.372Z  INFO 4539 --- [           main] o.s.batch.core.step.AbstractStep         : Executing step: [filePreparation]
   2026-08-21T12:49:49.396Z  INFO 4539 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [filePreparation] executed in 32ms
   2026-08-21T12:49:49.444Z  INFO 4539 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Duplicate step [filePreparation] detected in execution of job=[BillingJob]. If either step fails, both will be executed again on restart.
   2026-08-21T12:49:49.460Z  INFO 4539 --- [           main] o.s.batch.core.step.AbstractStep         : Executing step: [filePreparation]
   2026-08-21T12:49:49.470Z  INFO 4539 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [filePreparation] executed in 15ms
   2026-08-21T12:49:49.504Z  INFO 4539 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{JobParameter{name='input.file', value=src/main/resources/billing-2026-01.csv, type=class java.lang.String, identifying=true}}] and the following status: [COMPLETED] in 167ms
   ...
   ```

   This means the job was completed successfully. But, it's always good to verify the results are as expected.

2. Verify the job succeeded.

   Let's check the content of the `BILLING_DATA` table. In the **Terminal**, run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select count(*) from BILLING_DATA;'
   ```

   You should see that the table contains 1000 records, which is the number of lines in the input file.

It worked!
