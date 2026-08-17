As we discussed in the related lesson, our Billing Job has a flaw: it is hard-coded to only process one data input file. But, we can update our job to be dynamically configured with an input file at runtime.

1. Review the hard-coded input file.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` and take a look at the `billingDataFileReader` bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public FlatFileItemReader<BillingData> billingDataFileReader() {
      return new FlatFileItemReaderBuilder<BillingData>()
            .name("billingDataFileReader")
            .resource(new FileSystemResource("staging/billing-2023-01.csv")) // <== hard-coded input file!
           ...
   }
   ```

   As you can see, `staging/billing-2023-01.csv` will be the only input file that ever gets processed. Let's make that more flexible!

1. Update the file reader.

   Our file readers needs to be a `@StepScope` bean and read the `input.file` at runtime.

   Update the `billingDataFileReader` bean definition as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "billingDataFileReader"
   description: "Open billingDataFileReader()"
   ```

   ```java
   @Bean
   @StepScope
   public FlatFileItemReader<BillingData> billingDataFileReader(@Value("#{jobParameters['input.file']}") String inputFile) {
      return new FlatFileItemReaderBuilder<BillingData>()
            .name("billingDataFileReader")
            .resource(new FileSystemResource(inputFile))
            .delimited()
            .names("dataYear", "dataMonth", "accountId", "phoneNumber", "dataUsage", "callDuration", "smsCount")
            .targetType(BillingData.class)
            .build();
   }
   ```

   You should also add the following import statements:

   ```java
   import org.springframework.batch.core.configuration.annotation.StepScope;
   import org.springframework.beans.factory.annotation.Value;
   ```

   In this snippet, we made the `billingDataFileReader` step-scoped in order to configure it with the `input.file` job parameter. The value is not hard-coded any more! We can now pass any file as a job parameter and it will be picked up by this reader.

We've fixed the file reader. Next, let's fix how data is pulled in from the database.
