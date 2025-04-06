The last thing we have to do is to update the test of our job to accept the new job parameters.

1. Add parameters to the test.

   Open `src/test/java/example/billingjob/BillingJobApplicationTests.java` and update the "given" part of the `testJobExecution` method as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "JobParameters jobParameters = new JobParametersBuilder()"
   description: "Open testJobExecution()"
   ```

   ```java
   // given
   JobParameters jobParameters = new JobParametersBuilder()
   		.addString("input.file", "input/billing-2023-01.csv")
   		.addString("output.file", "staging/billing-report-2023-01.csv")
   		.addJobParameter("data.year", 2023, Integer.class)
   		.addJobParameter("data.month", 1, Integer.class)
   		.toJobParameters();
   ```

   In this update, we added the three new job parameters: `output.file`, `data.year` and `data.month`. The test execution and assertions are not impacted by this change. Let's verify the test. In the **Terminal**, run the following command:

1. Run the test.

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
   ```

   The test should pass, which means our step-scoped components are correctly configured dynamically at runtime with the new job parameters!
