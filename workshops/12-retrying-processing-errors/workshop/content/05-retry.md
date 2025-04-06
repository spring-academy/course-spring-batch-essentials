Pricing exceptions are transient. In fact, retrying a failed pricing operation might succeed in a subsequent attempt. Therefore, `PricingException` is a good candidate to declare as a retryable exception. Let's do that.

1. Add the retry feature.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` and update the definition of `step3` as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "public Step step3"
   description: "Open step3()"
   ```

   ```java
   @Bean
   public Step step3(JobRepository jobRepository, JdbcTransactionManager transactionManager,
   						   ItemReader<BillingData> billingDataTableReader,
   						   ItemProcessor<BillingData, ReportingData> billingDataProcessor,
   						   ItemWriter<ReportingData> billingDataFileWriter) {
   	return new StepBuilder("reportGeneration", jobRepository)
   			.<BillingData, ReportingData>chunk(100, transactionManager)
   			.reader(billingDataTableReader)
   			.processor(billingDataProcessor)
   			.writer(billingDataFileWriter)
   			.faultTolerant()
   			.retry(PricingException.class)
   			.retryLimit(100)
   			.build();
   }
   ```

   That's a lot of code! Let's understand the changes:

   - First, we have made the report generation step tolerant to faults by calling the `StepBuilder.faultTolerant()` method.
   - After that, we declared the `PricingException` as a retryable exception using the `.retry(PricingException.class)` call.
   - Finally, we defined a retry limit of 100

   With this configuration, any `process` operation that throws a `PricingException` will be retried at most 100 times, after which the step will be marked as failed.

   Let's try the job again.

1. Build and run the job again.

   First, we need to build the new version of the job. Open a **Terminal** and run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Now run the job again:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-04.csv output.file=staging/billing-report-2023-04.csv skip.file=staging/billing-data-skip-2023-04.psv data.year=2023 data.month=4
   ```

   The job should now succeed!

Failed operations will be _retried without failing the entire job_.

That's great compared to the previous version where we had to retry the job manually again and again.
