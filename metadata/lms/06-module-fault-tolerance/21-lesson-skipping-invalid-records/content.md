While transient errors can be addressed by retrying operations, some errors are "permanent". For example, if a line in a flat file is not correctly formatted and the item reader can't parse it, no matter how many times you retry the read operation, it'll always fail.

So, how do we address these kinds of errors? Should we let the job fail until we fix the input file and restart the job afterwards? Or, can we just skip that line and continue processing the rest of the file?

In the previous lesson and Lab, you learned how to fix the data and restart a failed job instance. In this lesson, you'll learn how to skip incorrect items.

## Skipping Incorrect Items

In the previous lesson and Lab, you've seen an example of how you can fix the input data of a batch job and restart it until completion. In fact, we had to fix a couple of incorrect lines and restart the job twice.

While this was feasible because only a couple of lines were incorrect, what if dozens of input lines are incorrect? Are we going to fix lines one by one and restart the job dozens of times? This is obviously inefficient, and for this reason, Spring Batch provides a feature to skip bad items for later analysis.

The skip feature in Spring Batch is specific to chunk-oriented steps. This feature covers all of the phases of the chunk-oriented processing model: when errors occur while reading, processing, or writing items.

To activate this feature, you'll need to define a "fault-tolerant" step.

### Fault-Tolerant Step

As mentioned in previous lessons, the main entry point to create steps is the `StepBuilder` API. Creating a fault-tolerant step is done by calling the `faultTolerant()` method on the `StepBuilder`.

Here's an example:

```java
@Bean
public Step step(
   JobRepository jobRepository, JdbcTransactionManager transactionManager,
   ItemReader<String> itemReader, ItemWriter<String> itemWriter) {
   return new StepBuilder("myStep", jobRepository)
		.<String, String>chunk(100, transactionManager)
		.reader(itemReader)
		.writer(itemWriter)
		.faultTolerant()
		.skip(FlatFileParseException.class)
		.skipLimit(5)
		.build();
}
```

This method returns a `FaultTolerantStepBuilder` that allows you to define fault-tolerance features (skip and retry). In this case, we're defining a skip policy.

The `skip()` method defines which exception should cause the current item to be skipped. In this example, if a `FlatFileParseException` occurs (most likely during the read operation because the current line cannot be parsed), the current line in the flat-file will be skipped.

But, what does it mean for an item to be skipped?

### Skipped Items

An item being skipped simply means that it'll be excluded from the current chunk. Spring Batch won't fail the step immediately, rather, it'll continue processing the next item.

### Skip Limit

Finally, the `skipLimit()` method is used to define the maximum number of items to skip. We might tolerate a reasonable number of items to skip, but if this number is high, then something is fundamentally wrong with the input data, and it would be better to let the step fail, then analyze the problem in detail.

## Handling Skipped Items

Skipped items can be audited by using a `SkipListener`.

The `SkipListener` is an extension point that Spring Batch provides to give the developer a way to handle skipped items, like logging them or saving them somewhere for later analysis.

The `SkipListener` is defined as follows:

```java
public interface SkipListener<T, S> extends StepListener {

   default void onSkipInRead(Throwable t) { }

   default void onSkipInWrite(S item, Throwable t) { }

   default void onSkipInProcess(T item, Throwable t) { }

}
```

This interface provides 3 methods to implement the logic of what to do if an item is skipped during the `read`, `process` or `write` operation. Once implemented, the `SkipListener` can be registered in the step using the `FaultTolerantStepBuilder.listener(SkipListener)` API.

We'll see an example of how to implement this interface and use it in the Lab for this Lesson.

## Custom Skip Policies

The skip policy we've seen earlier is based on an Exception. You declare which exception should happen to skip the current item. But what if the decision to skip an item isn't (only) based on an exception? This is where the `SkipPolicy` interface comes into play.

Spring Batch provides a strategy interface called `SkipPolicy` that allows you to provide a custom skip policy that's based on a custom business rule. The `SkipPolicy` interface is defined as follows:

```java
@FunctionalInterface
public interface SkipPolicy {

	boolean shouldSkip(Throwable t, long skipCount) throws SkipLimitExceededException;

}
```

This is a functional interface with a single method `shouldSkip` that's designed to specify whether the current item should be skipped or not. This method provides a handle to a `Throwable` object that contains all the details and context about the current item and the exception that happened.

A typical example of a "skippable" exception is the `FlatFileParseException` that we saw earlier, which provides the current item along with the line number in the input file.

Once the custom `SkipPolicy` is implemented, it can be registered in the step using the `FaultTolerantStepBuilder.skipPolicy(SkipPolicy)` API.
