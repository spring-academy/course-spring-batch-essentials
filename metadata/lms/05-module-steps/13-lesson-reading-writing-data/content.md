In the previous Lab, we implemented our first step to copy a file from one directory to another. We did that by creating a custom `Tasklet` and using it within a `TaskletStep`. Now, the second step of our job consists of reading the billing data from that file and writing it into a database table. While we could also do that by implementing a custom `Tasklet`, we'll have to write a lot of boilerplate code to read the file content, parse it, and write it to the database.

Fortunately, Spring Batch provides APIs to take care of all of the above mentioned toil for us in a configurable and reusable manner!

In this lesson, we'll cover those APIs and see how to use them to read and write data to a variety of item-oriented datasources, like files and databases.

## Understanding the Chunk-Oriented Processing Model

Ingesting a file into a database table seems like a simple and easy task at first glance, but this simple task is actually quite challenging! What if the input file is big enough that it doesn't fit in memory? What if the process that ingests the file is terminated abruptly half way through? How do we deal with these situations in an efficient and fault-tolerant way?

Spring Batch comes with a processing model that is designed and implemented to address those challenges. It is called the _chunk-oriented processing model_. The idea of this model is to process the datasource in _chunks_ of a configurable size.

A chunk is a collection of "items" from the datasource. An item can be a line in a flat file, a record in a database table, etc.

A chunk of items is represented by the `Chunk<T>` API, which is a thin wrapper around a list of objects of type `T`. The generic type `T` represents the type of items. This means items can be of any type.

```java
public class Chunk<T> implements Iterable<T> {

   private List<T> items;

   // methods to add and get items
}
```

### Transactions

Each chunk of items is read and written within the scope of a transaction. This way, chunks are either committed together or rolled back together, meaning they either all succeed or fail together. You can think of this as an "all-or-nothing" approach, but only for the specific chunk that's being processed.

If the transaction is committed, Spring Batch records, as part of the transaction, the execution progress (read count, write count, etc) in its metadata repository and uses that information to restart where it left off in case of a failure.

If an error occurs while processing a chunk of items, then the transaction will be rolled back by the framework. Therefore, the execution state won't be updated and Spring Batch would restart from the last successful save-point in case of failure – kind of like a video game!

The number of items to include in a chunk is called the _commit-interval_, which is the configurable size of a chunk that should be processed within the scope of a single transaction.

In the next section, we'll explore the APIs to read and write items in chunks that are provided by Spring Batch.

## Reading Data

Reading data from a flat file is different from reading data from a database table or from a messaging broker queue. For this reason, Spring Batch provides a strategy interface to read items in a consistent and implementation-agnostic way.

### The `ItemReader` Interface

Reading data in Spring Batch is done through the `ItemReader` interface, which is defined as follows:

```java
@FunctionalInterface
public interface ItemReader<T> {
   @Nullable
   T read() throws Exception;
}
```

This functional interface provides a single method named `read` to read one piece of data, or an item. Each call to this method is expected to return a single item, one at a time.

Spring Batch will call `read` as needed to create chunks of items. This way, Spring Batch never loads the entire datasource in memory, only chunks of data.

The type of items `T` is generic, so it's up to the implementation to decide which type of items is returned.

### Error Handling

If an error occurs while reading an item, implementations are expected to throw an exception to signal the error to the framework.

Note how the method `read` is annotated with `@Nullable`, which means it might return `null`.

```java
@Nullable
T read() throws Exception;
```

Returning `null` from that method is the way to signal to the framework that the datasource is exhausted. Remember, batch processing is about processing finite data sets, not infinite streams of data.

### The `ItemReader` Library

Spring Batch comes with a large library of `ItemReader` implementations to read data from a variety of datasources, like files, databases, message brokers, etc. As a Spring Batch developer, you'll typically just have to configure one of these readers and use it in a step.

In this lesson and its Lab, we'll read data from a flat file, so we'll use the `FlatFileItemReader` which is designed for just that. Here is an example of how to configure such a reader:

```java
@Bean
public FlatFileItemReader<BillingData> billingDataFileReader() {
    return new FlatFileItemReaderBuilder<BillingData>()
            .name("billingDataFileReader")
            .resource(new FileSystemResource("staging/billing-data.csv"))
            .delimited()
            .names("dataYear", "dataMonth", "accountId", "phoneNumber", "dataUsage", "callDuration", "smsCount")
            .targetType(BillingData.class)
            .build();
}
```

In this example, we create a bean of type `FlatFileItemReader` which returns items of type `BillingData`. This type represents one item from the billing data file, which we'll explain and create in the Lab of this lesson. To build a `FlatFileItemReader`, we'll use the `FlatFileItemReaderBuilder` API to specify the path of the input file, the expected fields in that file, and the target type to map data to. We'll explain all these details in the Lab for this lesson. All you have to understand for now is that leveraging one of the built-in readers really boils down to configuring an instance of it like we did in this snippet.

Unless you have a very specific requirement to implement a custom item reader, you should be able to leverage one of the item readers provided by Spring Batch out-of-the-box, as shown above. In a future lesson, we'll use the `JdbcCursorItemReader` to read data from a database table using the JDBC API. You can find the list of all available readers in the "Links" section of the lesson.

## Writing Data

Similar to the `ItemReader` interface, writing data with Spring Batch is done through the `ItemWriter` interface, which is defined as follows:

```java
@FunctionalInterface
public interface ItemWriter<T> {

   void write(Chunk<? extends T> chunk) throws Exception;
}
```

### Writing Chunks

The `write` method expects a chunk of items. _Unlike_ reading items which is done one item at a time, writing items is done in chunks. The reason for this is that bulk writes are typically more efficient than single writes.

For example, the `JdbcBatchItemWriter`, which is designed to write items in a relational database table, leverages the JDBC Batch API to insert items in batch mode. This would not have been possible if the `ItemWriter` interface was designed to write one item at a time.

Spring Batch provides several implementations of the `ItemWriter` interface to write data to a variety of targets like files, databases and message brokers. You'll find the list of available writers in the "Links" section of this lesson.

Note: if an error occurs while writing items, implementations of this interface are expected to throw an exception to signal the problem to the framework.

### `ItemWriter` Example

For the first step of our Billing job, we need to write data to a relational database table, so we'll use a `JdbcBatchItemWriter` for that. Here's an example of how to configure such a writer:

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

We'll go over this snippet in detail in the Lab. For now, make note of how the `JdbcBatchItemWriter` is built using database-specific constructs such as a SQL statement and a `DataSource`. Consider that other writers, such as `FlatFileItemWriter`, would be built using constructs that are specific to those implementations.

## Configuring Chunk-Oriented Tasklet Steps

Now that we've explained how to read and write data in Spring Batch using item readers and writers, it's time to use those readers and writers to define a chunk-oriented step.

A chunk-oriented step in Spring Batch is a `TaskletStep` configured with a specific `Tasklet` type, the `ChunkOrientedTasklet`. This `Tasklet` is what implements the chunk-oriented processing model (explained above) using an item reader and an item writer.

Similar to configuring a `TaskletStep` with a custom tasklet, configuring a chunk-oriented tasklet is also done through the `StepBuilder` API, except we'll need to call the `chunk` method instead of the `tasklet` method.

So let's see how to use this API to create the second step of our job, the file ingestion step.

### File Ingestion

The file ingestion step is intended to read billing data from a flat file and write it into a relational database, so assuming the `FlatFileItemReader<BillingData>` and `JdbcBatchItemWriter<BillingData>` that we showed in previous sections are in place, here's how to configure the step:

```java
@Bean
public Step ingestFile(
              JobRepository jobRepository,
              PlatformTransactionManager transactionManager,
              FlatFileItemReader<BillingData> billingDataFileReader,
              JdbcBatchItemWriter<BillingData> billingDataTableWriter) {

    return new StepBuilder("fileIngestion", jobRepository)
            .<BillingData, BillingData>chunk(100, transactionManager)
            .reader(billingDataFileReader)
            .writer(billingDataTableWriter)
            .build();
}
```

As we've seen before, we'll need to specify the step name and the job repository. This is common for all step types. Since we're creating a chunk-oriented `TaskletStep`, we'll need to provide two parameters:

- The first parameter is the chunk-size, or the commit-interval. In this case, it is set to 100, which means Spring Batch will read and write 100 items as a unit in each transaction.
- The second parameter is a reference to a `PlatformTransactionManager` to manage the transactions of the tasklet. Remember, each call the `Tasklet.execute` is done within the scope of a transaction.

Finally, the `StepBuilder.chunk` API guides the user to further configure the chunk-oriented tasklet, by specifying the item reader and writer to use through the `reader` and `writer` methods respectively.

That's all it takes to configure a chunk-oriented tasklet in Spring Batch. In the next Lab, we'll configure this step, and add it as a second step to our job.
