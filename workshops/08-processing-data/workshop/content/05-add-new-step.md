The report generation step is a chunk-oriented step, similar to the first step of our job. The main difference here is that we are adding an item processor, the `billingDataProcessor`.

1. Define the report generation step.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
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
               .build();
   }
   ```

   You should also add the following import statement:

   ```java
   import org.springframework.batch.item.ItemProcessor;
   ```

   That's a lot of parameters and method calls! Let's make sure we understand exactly what's going on.

1. Understand the report generation step.

   We have declared a bean named `step3` of type `Step`, which represents the final step, `reportGeneration`.

   Similar to the previous chunk-oriented step, `step2` named `fileIngestion`, we need to pass a reference to the job repository and transaction manager. The chunk size is also 100, which is a good starting value.

   The item reader, processor and writer are also passed as parameters to the bean definition method so they are autowired by Spring from the previous bean definitions.

   Let's now make sure we understand what each of these three autowired beans is doing, and in which phase they are being used.

   - ```java
     .reader(billingDataTableReader)
     ```

     This phase is using our `JdbcCursorItemReader` bean to read and map data that has already been inserted into the database in `step2`.

   - ```java
     .processor(billingDataProcessor)
     ```

     This is our fancy new `ItemProcessor` bean that not only filters out bills less than $150, but also enriches the data with the actual spending amount.

     There is a big difference compared to the `fileIngestion` in `step2`, which did not have a `processor` phase. While the `fileIngestion` step did not change the item type (since it did not use an item processor), notice how the item type changes here: `<BillingData, ReportingData>chunk(...)`. In fact, this final step _changes the item type with an item processor_, hence this notation.

   - ```java
     .writer(billingDataFileWriter)
     ```

     Finally, we use a `FlatFileItemWriter` to write the filtered, enriched data to a file for reporting purposes.

1. Add the third step to the job flow.

   In the `src/main/java/example/billingjob/BillingJobConfiguration.java` file, update the `job` bean definition as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "public Job job"
   description: "Open job()"
   ```

   ```java
   @Bean
   public Job job(JobRepository jobRepository, Step step1, Step step2, Step step3) {
   	return new JobBuilder("BillingJob", jobRepository)
   			.start(step1)
   			.next(step2)
   			.next(step3)
   			.build();
   }
   ```

   By adding `step3` after `step2` in `JobBuilder`, the `reportGeneration` step will be executed only after the `fileIngestion` step has completed successfully.

We've set everything up. Time for us to run the job!
