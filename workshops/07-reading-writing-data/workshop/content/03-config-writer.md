Billing data should be saved into the database for later use to generate the billing report. For this, we need to create a table in the database to store the billing data.

1. Review database structure.

   Here is the table definition where the billing data will be stored:

   ```java
   create table BILLING_DATA
   (
       DATA_YEAR     INTEGER,
       DATA_MONTH    INTEGER,
       ACCOUNT_ID    INTEGER,
       PHONE_NUMBER  VARCHAR(12),
       DATA_USAGE    FLOAT,
       CALL_DURATION INTEGER,
       SMS_COUNT     INTEGER
   );
   ```

   You can find this table definition in the `src/sql/schema-billing.sql` file.

   ```editor:open-file
   file: ~/exercises/src/sql/schema-billing.sql
   description: "Open schema-billing.sql"
   ```

   You would normally need to create this table in the database beforehand, but this Lab has been configured with that table already in place to simplify this task for you. Let's check this by counting how many records in that table.

   Open the **Terminal**, run the following command and note the results:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select count(*) from BILLING_DATA;'
    count
   -------
        0
   (1 row)
   ```

   The table should be empty at this point, and we expect to see some records in it later when we will run the job. Now that the target table is in place, it is time to configure the item writer.

1. Create the `billingDataTableWriter`.

   Since we are writing data to a JDBC datasource, the most convenient item writer to use is the `JdbcBatchItemWriter<T>`. This item writer is designed to write items to a database using the JDBC API. Let's configure such a writer.

   In the Editor, open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public JdbcBatchItemWriter<BillingData> billingDataTableWriter(DataSource dataSource) {
       String sql = "insert into BILLING_DATA values (:dataYear, :dataMonth, :accountId, :phoneNumber, :dataUsage, :callDuration, :smsCount)";
       return new JdbcBatchItemWriterBuilder<BillingData>()
               .dataSource(dataSource)
               .sql(sql)
               .beanMapped()
               .build();
   }
   ```

   You should also add the following import statements:

   ```java
   import javax.sql.DataSource;
   import org.springframework.batch.item.database.JdbcBatchItemWriter;
   import org.springframework.batch.item.database.builder.JdbcBatchItemWriterBuilder;
   ```

   The `JdbcBatchItemWriter` needs a lot of information to correctly write our billing data to the database. Let's look at the `JdbcBatchItemWriter` in more detail for how this is accomplished.

1. Understand the `JdbcBatchItemWriter`.

   The `JdbcBatchItemWriter` needs to know the target database to write data to and the SQL statement to execute.

   In the previous snippet, we define a bean named `billingDataTableWriter` of type `JdbcBatchItemWriter<BillingData>`. We pass the datasource as a parameter to the method, which represents the target database into which data should be stored.

   We also define the SQL `insert` statement that the writer should invoke to insert items. This statement specifies the target table `BILLING_DATA` which we created earlier, and also the list of columns to insert. Note how the list of columns (`:dataYear`, `:dataMonth`, etc) corresponds to the names of fields in the `BillingData` type.

   But how would the item writer bind data from `BillingData` objects created by the reader to the columns of the table using the `:fieldName` syntax?

   This is where the call to the `.beanMapped()` method comes to play. This method instructs the writer to use the Java Reflection API to call getter methods to get the value of each field with the _same_ name as the database column. For example, to bind the `:dataYear` column in the SQL query, the writer will call the `dataYear()` on the `BillingData` instance of the current item.

That's all for the item writer. We can now define the chunk-oriented step.
