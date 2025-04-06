Now that the item reader and writer have been configured, we can define the `fileIngestion` step.

1. Define the second step.

   In the editor, open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public Step step2(
      JobRepository jobRepository, JdbcTransactionManager transactionManager,
      ItemReader<BillingData> billingDataFileReader, ItemWriter<BillingData> billingDataTableWriter) {
   	return new StepBuilder("fileIngestion", jobRepository)
   			.<BillingData, BillingData>chunk(100, transactionManager)
   			.reader(billingDataFileReader)
   			.writer(billingDataTableWriter)
   			.build();
   }
   ```

   Be sure to add the two additional imports:

   ```java
   import org.springframework.batch.item.ItemReader;
   import org.springframework.batch.item.ItemWriter;
   ```

   In this snippet, we define a bean named `step2` of type `Step`, and it has a lot going on. Let's dive into the details next.

1. Understand the file ingestion step definition.

   As we have seen previously, all step types require at least the job repository and the step name. In this case, the job repository is passed as a parameter to the bean definition method and the step name is `fileIngestion`.

   Now since we are creating a chunk-oriented step, which is a `TaskletStep`, we need also to pass a reference to a transaction manager and to specify the chunk size, which is 100 in this case. This is done by calling the `.chunk(...)` method.

   The syntax `<BillingData,BillingData>chunk(...)` is used to tell Spring Batch that the input and output of the step are of type `BillingData`, meaning that the reader will return items of type `BillingData` and that the writer will write items of type `BillingData` as well.

   In other words, this step does not change the type of items during its execution. It is possible to change the item type if data should be transformed during the processing. We will cover this in the next lesson.

   **Heads-up:** The value of the chunk-size depends heavily on the use case and should be set in an empirical way. The value 100 is usually a good starting point, but this might not always be the case.

   Finally, we set the item reader and writer using the `.reader()` and `.writer()` methods respectively. The reader and writer are passed as parameters to the bean definition method, which means they will be autowired by Spring from the bean definitions we configured in the previous sections.

Now that everything is configured correctly we're ready to run this new version of our job!
