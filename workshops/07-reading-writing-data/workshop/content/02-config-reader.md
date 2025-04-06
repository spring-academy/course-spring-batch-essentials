Since we are dealing with a flat file as input, the most convenient reader to use is the `FlatFileItemReader<T>`. This reader is designed to read data from any flat file format like delimited files (CSV, TSC, etc) or fixed-length files.

Now the question is: what type of items `T` should this reader return?

While we can configure the reader to return Strings or byte arrays from the file, this would be inconvenient to work with while processing the data. As a Spring Batch developer, what we typically do is define a domain type that represents the data being processed. In the case of the Spring Cellular domain, we will define a type named `BillingData` that represents a line from the input file.

1. Create the domain type.

   In the Editor, create a file named `BillingData.java` in the `src/main/java/example/billingjob` folder with the following content:

   ```editor:append-lines-to-file
   file: ~/exercises/src/main/java/example/billingjob/BillingData.java
   description: "Create BillingData.java"
   ```

   ```java
   package example.billingjob;

   public record BillingData (
   		int dataYear,
   		int dataMonth,
   		int accountId,
   		String phoneNumber,
   		float dataUsage,
   		int callDuration,
   		int smsCount) {
   }
   ```

   This type is a Java record that represents a record (or a line) from the input file.

   Fields in the `BillingData` type correspond to columns in the input file. The declaration order of columns does not matter here, we will define it later when we configure the reader.

   Now that we defined the type of items, we can proceed to configuring the item reader.

1. Create the `billingDataFileReader`.

   In the Editor, open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public FlatFileItemReader<BillingData> billingDataFileReader() {
       return new FlatFileItemReaderBuilder<BillingData>()
               .name("billingDataFileReader")
               .resource(new FileSystemResource("staging/billing-2023-01.csv"))
               .delimited()
               .names("dataYear", "dataMonth", "accountId", "phoneNumber", "dataUsage", "callDuration", "smsCount")
               .targetType(BillingData.class)
               .build();
   }
   ```

   You should also add the following import statements:

   ```java
   import org.springframework.batch.item.file.FlatFileItemReader;
   import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
   import org.springframework.core.io.FileSystemResource;
   ```

   In this snippet, we define a bean named `billingDataFileReader` of type `FlatFileItemReader<BillingData>`. We use the `FlatFileItemReaderBuilder` to create the reader and we specify a few properties such as the reader's name and the input file `staging/billing-2023-01.csv`.

   We also tell the reader that the input file is expected to be delimited (using the `.delimited()` method) and that columns are expected to be defined in a specific order, which corresponds to the list of fields (`.names(...)`) in the target type `BillingData`.

   With that in place, the item reader will read lines one by one, and will create a new instance of `BillingData` for each line.

Note how with just a few lines of configuration code, Spring Batch actually saved us a lot of boilerplate code for reading the input file (opening/closing the file), reading the file line by line, parsing fields and mapping data to corresponding types in `BillingData`. This is a huge amount of tedious and error-prone work that Spring Batch saved us from doing!

Now that we defined the item reader, let's move on and define the item writer.
