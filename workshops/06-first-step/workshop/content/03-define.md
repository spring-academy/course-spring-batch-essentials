Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

```editor:open-file
file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
description: "Open BillingJobConfiguration.java"
```

```java
@Bean
public Step step1(JobRepository jobRepository, JdbcTransactionManager transactionManager) {
	return new StepBuilder("filePreparation", jobRepository)
			.tasklet(new FilePreparationTasklet(), transactionManager)
			.build();
}
```

You should also add the following import statements:

```java
import org.springframework.batch.core.Step;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.jdbc.support.JdbcTransactionManager;
```

In this snippet, we define a bean named `step1` of type `Step`.

We pass a `JobRepository` for the step and a `JdbcTransactionManager` for the tasklet. This transaction manager is auto-configured by Spring Boot and we can use it here to define the `TaskletStep`.

**Remember**, a `TaskletStep` requires a transaction manager to manage the transaction around each iteration of the `Tasklet`. The `TaskletStep` is defined by the call to `StepBuilder.tasklet` to which we pass an instance of the `FilePreparationTasklet` and the transaction manager to drive transactions.
