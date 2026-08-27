Some errors are transient by nature, such as calling a flaky web service or hitting a database lock. Retrying the same operation when such errors occur might succeed in a subsequent attempt. It would be unfortunate and inefficient to fail an entire job and have to restart it later if one could just retry an operation.

For this reason, Spring Batch provides a retry feature that lets you retry operations that might encounter a transient error. This lesson is about the retry feature.

## Retrying Transient Errors

The retry feature in Spring Batch is built directly on Spring Framework's own retry support (the `org.springframework.core.retry` package), introduced in Spring Framework 7. Previously, Spring Batch relied on the standalone [Spring Retry](https://github.com/spring-projects/spring-retry) library for this feature, but that library is no longer actively maintained, so its functionality has now been absorbed into Spring Framework itself.

Similar to the skip feature, the retry feature is designed for chunk-oriented steps, specifically for the processing and writing phases. The reading phase is _not_ retryable.

To activate the retry feature, you'll need to define a "fault-tolerant" step. As mentioned in previous lessons, chunk-oriented steps are created with the `ChunkOrientedStepBuilder` API, and creating a fault tolerant step is done by calling the `faultTolerant()` method on that same builder.

Here's an example:

```java
@Bean
public Step step(
   JobRepository jobRepository, JdbcTransactionManager transactionManager,
   ItemReader<String> itemReader, ItemWriter<String> itemWriter) {
   return new ChunkOrientedStepBuilder<String, String>("myStep", jobRepository, 100)
		.reader(itemReader)
		.writer(itemWriter)
		.transactionManager(transactionManager)
		.faultTolerant()
		.retry(TransientException.class)
		.retryLimit(5)
		.build();
}
```

In this snippet, the chunk-oriented step, `myStep`, is declared as a fault-tolerant step, thanks to the call to the `.faultTolerant()` method. Calling `faultTolerant()` on the `ChunkOrientedStepBuilder` unlocks its fault-tolerance features (skip and retry).

In this case, we're defining a retry policy as follows:

> Any `TransientException` (or one of its subclasses) should be retried at most 5 times, after which the step should be marked as failed.

The exception to retry is defined with the `.retry()` method, while the retry limit is defined with the `retryLimit()` method.

We'll implement a retry feature in the Lab of this lesson.

## Handling Retry Attempts

For auditing purposes, Spring Batch provides a way to register a `RetryListener` in the step in order to plug in custom code around retry attempts: `beforeRetry`, `onRetrySuccess`, `onRetryFailure`, and so on.

The `RetryListener` API is part of Spring Framework's `org.springframework.core.retry` package and is defined as follows:

```java
public interface RetryListener {

   default void beforeRetry(RetryPolicy retryPolicy, Retryable<?> retryable, RetryState retryState) {
   }

   default void onRetrySuccess(RetryPolicy retryPolicy, Retryable<?> retryable, Object result) {
   }

   default void onRetryFailure(RetryPolicy retryPolicy, Retryable<?> retryable, Throwable throwable) {
   }

   default void onRetryPolicyExhaustion(RetryPolicy retryPolicy, Retryable<?> retryable, RetryException retryException) {
   }

   // plus onRetryableExecution, onRetryPolicyInterruption, and onRetryPolicyTimeout
}
```

This interface is an extension point that gives the developer a way to execute custom code before a retry attempt (`beforeRetry`), after a successful attempt (`onRetrySuccess`), after a failed attempt (`onRetryFailure`), or once the retry policy gives up entirely (`onRetryPolicyExhaustion`). Every method has a default no-op implementation, so you only need to override the ones you actually care about.

Typical examples of using this API are logging and reporting retry operations. See the "Links" section for more details about this API.

Once you've implemented the `RetryListener`, you can register it in the step by using the `ChunkOrientedStepBuilder.retryListener(RetryListener)` method.

## Custom Retry Policies

Similar to the `SkipPolicy` API for custom skip policies, Spring Batch lets you define custom retry policies by implementing the `RetryPolicy` interface. The `RetryPolicy` interface is part of Spring Framework's `org.springframework.core.retry` package and is defined as follows:

```java
public interface RetryPolicy {

   boolean shouldRetry(Throwable throwable);

   default Duration getTimeout() {
      // no timeout, by default
   }

   default BackOff getBackOff() {
      // a sensible default BackOff
   }

}
```

This interface is an extension point that lets you decide, based on the `Throwable` that was thrown, whether an operation should be retried (`shouldRetry`), how long to keep retrying before giving up altogether (`getTimeout`), and how long to wait between attempts (`getBackOff`, using Spring's `BackOff` abstraction, such as a `FixedBackOff` or an `ExponentialBackOff`).

Rather than implementing this interface from scratch, you'll typically build one using the fluent `RetryPolicy.builder()` API:

```java
RetryPolicy retryPolicy = RetryPolicy.builder()
      .includes(PricingException.class)
      .maxRetries(100)
      .delay(Duration.ofSeconds(1))
      .multiplier(2.0)
      .maxDelay(Duration.ofSeconds(30))
      .build();
```

Once built, a `RetryPolicy` can be registered directly on a step with `ChunkOrientedStepBuilder.retryPolicy(RetryPolicy)`, giving you full control over backoff and timeout behavior beyond what the simpler `.retry()` and `.retryLimit()` methods offer.
