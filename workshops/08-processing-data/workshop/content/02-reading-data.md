Since we need to read data from a JDBC datasource, the most convenient reader that Spring Batch provides is the `JdbcCursorItemReader`.

This reader opens a JDBC cursor on a given table and returns items from the JDBC `ResultSet` returned by the cursor.

1. Create the `billingDataTableReader`.

   Open `src/main/java/example/billingjob/BillingJobConfiguration.java` class and add the following bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public JdbcCursorItemReader<BillingData> billingDataTableReader(DataSource dataSource) {
       String sql = "select * from BILLING_DATA";
       return new JdbcCursorItemReaderBuilder<BillingData>()
               .name("billingDataTableReader")
               .dataSource(dataSource)
               .sql(sql)
               .rowMapper(new DataClassRowMapper<>(BillingData.class))
               .build();
   }
   ```

   You also need to add the following import statements:

   ```java
   import org.springframework.batch.item.database.JdbcCursorItemReader;
   import org.springframework.batch.item.database.builder.JdbcCursorItemReaderBuilder;
   import org.springframework.jdbc.core.DataClassRowMapper;
   ```

1. Understand the new item reader.

   In this bean definition method, we use the `JdbcCursorItemReaderBuilder` to create a `JdbcCursorItemReader`. The main configuration properties for this reader are the datasource, the SQL query to fetch data, and the target item type to which data should be mapped.

   In our case, the target item type is `BillingData`, which is a Java record.

   Now the question is: how to map database row column values to fields in the `BillingData` type? For that, we can use the `DataClassRowMapper` from Spring Framework.

   The `DataClassRowMapper` class uses the same reflection technique in `JdbcBatchItemWriter.beanMapped` that we saw earlier in this course to map data from database columns to fields in the target type `BillingData`. We use an instance of `DataClassRowMapper` by calling the `.rowMapper(new DataClassRowMapper<>(BillingData.class))` method.

Now that we have a new item reader, let's proceed to defining the processing logic of our step.
