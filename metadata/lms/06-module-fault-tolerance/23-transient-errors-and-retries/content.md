Some errors are transient by nature, such as calling a flaky web service or hitting a database lock. Retrying the same operation when such errors occur might succeed in a subsequent attempt. It would be unfortunate and inefficient to fail an entire job and have to restart it later if one could just retry an operation.

For this reason, Spring Batch provides a retry feature that lets you retry operations that might encounter a transient error. This lesson is about the retry feature.

## Retrying Transient Errors

The retry feature in Spring Batch is based on the [Spring Retry](https://github.com/spring-projects/spring-retry) library. Spring Retry was historically part of Spring Batch itself but was extracted as a separate library, which is now used by many other projects in the Spring portfolio.

Similar to the skip feature, the retry feature is designed for chunk-oriented steps, specifically for the processing and writing phases. The reading phase is _not_ retryable.

To activate the retry feature, you'll need to define a "fault-tolerant" step. As mentioned in previous lessons, the main entry point to create steps is the `StepBuilder` API, and creating a fault tolerant step is done by calling the `faultTolerant()` method.

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
		.retry(TransientException.class)
		.retryLimit(5)
		.build();
}
```

In this snippet, the chunk-oriented step, `myStep`, is declared as a fault-tolerant step, thanks to the call to the `.faultTolerant()` method. This method returns a `FaultTolerantStepBuilder` that allows you to define the fault-tolerance features (skip and retry).

In this case, we're defining a retry policy as follows:

> Any `TransientException` (or one of its subclasses) should be retried at most 5 times, after which the step should be marked as failed.

The exception to retry is defined with the `.retry()` method, while the retry limit is defined with the `retryLimit()` method.

We'll implement a retry feature in the Lab of this lesson.

## Handling Retry Attempts

For auditing purposes, Spring Batch provides a way to register a `RetryListener` in the step in order to plug in custom code during retry attempts: `onError`, `onSuccess`, and so on.

The `RetryListener` API is part of Spring Retry and is defined as follows:

```java
public interface RetryListener {

   default <T, E extends Throwable> void onSuccess(
          RetryContext context,
          RetryCallback<T, E> callback,
          T result) {
   }

   default <T, E extends Throwable> void onError(
          RetryContext context,
          RetryCallback<T, E> callback,
	   Throwable throwable) {
   }
}
```

This interface is an extension point that gives the developer a way to execute custom code during retry attempts (that is, during failed attempts), by implementing the `onError` method. You can execute custom code upon successful attempts by implementing the `onSuccess` method.

Typical examples of using this API are logging and reporting retry operations. See the "Links" section for more details about this API.

Once you've implemented the `RetryListener`, you can register it in the step by using the `FaultTolerantStepBuilder.listener(RetryListener)` method.

## Custom Retry Policies

Similar to the `SkipPolicy` API for custom skip policies, Spring Batch provides a way to use custom retry policies by implementing the `RetryPolicy` interface. The `RetryPolicy` interface is part of Spring Retry and is defined as follows:

```java
public interface RetryPolicy extends Serializable {
   boolean canRetry(RetryContext context);
   void registerThrowable(RetryContext context, Throwable throwable);
}
```

This interface is an extension point that lets the user utilize custom rules for retrying items. Spring Retry already provides several ready-to-use implementations of this interface, including `MaxAttemptsRetryPolicy`, `CircuitBreakerRetryPolicy`, and others. See the "Links" section for more details about these implementations.
