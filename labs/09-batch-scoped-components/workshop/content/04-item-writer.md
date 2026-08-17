The last component we need to scope is the `billingDataFileWriter`. This is the writer that generates the billing report as part of the `reportGeneration` step.

Similarly to the input file being hard-coded in our file reader, the output file in this writer's bean definition method is hard-coded.

1. Review the hard-coded output file.

   Once again, open `src/main/java/example/billingjob/BillingJobConfiguration.java` and this time take a look at the `billingDataFileWriter` bean definition.

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "billingDataFileWriter"
   description: "Open billingDataFileWriter()"
   ```

   ```java
   @Bean
   public FlatFileItemWriter<ReportingData> billingDataFileWriter() {
   	return new FlatFileItemWriterBuilder<ReportingData>()
   	.resource(new FileSystemResource("staging/billing-report-2023-01.csv")) // <== hard-coded value!
     ...
   }
   ```

   Like the input file, the output file should be configurable for more flexibility if we decide to change its name or location. So let's fix that.

1. Introduce the `output.file` job parameter.

   Update the `billingDataFileWriter` bean definition as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "billingDataFileWriter"
   description: "Open billingDataFileWriter()"
   ```

   ```java
   @Bean
   @StepScope
   public FlatFileItemWriter<ReportingData> billingDataFileWriter(
      @Value("#{jobParameters['output.file']}") String outputFile) {
   	return new FlatFileItemWriterBuilder<ReportingData>()
   			.resource(new FileSystemResource(outputFile))
   			.name("billingDataFileWriter")
   			.delimited()
   			.names("billingData.dataYear", "billingData.dataMonth", "billingData.accountId", "billingData.phoneNumber", "billingData.dataUsage", "billingData.callDuration", "billingData.smsCount", "billingTotal")
   			.build();
   }
   ```

   In this snippet, we updated `billingDataFileWriter` to be a step-scoped bean by adding the `@StepScope` annotation.

   This lets us late-bind the output file from a new job parameter called `output.file`.

   With that in place, we can now specify the output file dynamically through a job parameter.

That's all we need as step-scoped beans. Let's update the way we launch the job with the new parameters.
