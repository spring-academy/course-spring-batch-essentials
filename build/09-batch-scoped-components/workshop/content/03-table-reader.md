Now we are faced with another challenge: if we ingest data from two billing files for different periods (let's say `billing-2023-01.csv` and then `billing-2023-02.csv`) the file ingestion step will load all that data in the `BILLING_DATA` table. However, the `billingDataTableReader` is configured to read _all the data_ from that table. That's not good!

1. Review the hard-coded SQL.

   Open `src/main/java/example/billingjob/BillingJobConfiguration.java` and take a look at the `billingDataTableReader` bean definition.

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "billingDataTableReader"
   description: "Open billingDataTableReader()"
   ```

   ```java
   @Bean
   public JdbcCursorItemReader<BillingData> billingDataTableReader(DataSource dataSource) {
   	String sql = "select * from BILLING_DATA"; // <== no filtering!
   	return new JdbcCursorItemReaderBuilder<BillingData>()
         ...
   }
   ```

   The problem is that our `billingDataTableReader` bean, which is part of the report generation step, is currently configured to read _all data from that table_ without filtering the current period from the database.

   We need to update this SQL query to filter data for the current period we are processing, to correctly generate the monthly report. So let's fix that!

1. Update the table reader.

   Update the `billingDataTableReader` bean definition, as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "billingDataTableReader"
   description: "Open billingDataTableReader()"
   ```

   ```java
   @Bean
   @StepScope
   public JdbcCursorItemReader<BillingData> billingDataTableReader(
      DataSource dataSource,
      @Value("#{jobParameters['data.year']}") Integer year,
      @Value("#{jobParameters['data.month']}") Integer month) {
   	String sql = String.format("select * from BILLING_DATA where DATA_YEAR = %d and DATA_MONTH = %d", year, month);
   	return new JdbcCursorItemReaderBuilder<BillingData>()
   			.name("billingDataTableReader")
   			.dataSource(dataSource)
   			.sql(sql)
   			.rowMapper(new DataClassRowMapper<>(BillingData.class))
   			.build();
   }
   ```

   Compared to the previous version, we marked this reader as step-scoped by adding the `@StepScope` annotation.

   We also added two new job parameters: `data.year` and `data.month` of type `Integer`. Those parameters will be used to specify the period we are planning to process when launching the job instance.

   Note how the SQL query is now dynamically defined to select only the records of the current period.

   **_Note:_** You might wonder why we did not extract the year and month from the file name. While we could do that, it would be risky, as the file name might not contain that information or might provide it in a different notation. For this reason, it is safer to explicitly pass the year and month as parameters to the job.

Now that we've fixed all of the item readers, let's move on to the item writers.
