The file ingestion step is the step where billing data is parsed from the input file and where the `FlatFileParseException` is likely to happen. Therefore, this is the step where we will define the skip policy.

1. Add the skip policy to the file ingestion step.

   We've added new classes and beans, but they are not used anywhere. Let's put them to use.

   In the `Editor`, open the `src/main/java/example/billingjob/BillingJobConfiguration.java` and update the file ingestion step, `step2`, as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "public Step step2"
   description: "Open step2()"
   ```

   ```java
   @Bean
   public Step step2(
      JobRepository jobRepository, JdbcTransactionManager transactionManager,
          ItemReader<BillingData> billingDataFileReader,
          ItemWriter<BillingData> billingDataTableWriter,
          BillingDataSkipListener skipListener) {
   	return new StepBuilder("fileIngestion", jobRepository)
   			.<BillingData, BillingData>chunk(100, transactionManager)
   			.reader(billingDataFileReader)
   			.writer(billingDataTableWriter)
   			.faultTolerant()
   			.skip(FlatFileParseException.class)
   			.skipLimit(10)
   			.listener(skipListener)
   			.build();
   }
   ```

   You also need to add the following import statement:

   ```java
   import org.springframework.batch.item.file.FlatFileParseException;
   ```

1. Understand the update.

   Here are the impacts of our changes:

   - We marked the `fileIngestion` step as a fault-tolerant step with the call to the `.faultTolerant()` method.
   - After that, we specified the `FlatFileParseException` as a skippable exception, with a skip limit of 10 items at most.
   - Finally, we registered the `BillingDataSkipListener` as a listener in the step.

     This listener is passed as a parameter to the step definition method, which means it will be autowired by Spring from the `skipListener` bean definition method defined earlier.

That's all it takes to make the file ingestion step tolerant to faults! Let's run the job and check the results.
