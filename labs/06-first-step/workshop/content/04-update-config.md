Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and replace the `job` bean as follows:

```editor:select-matching-text
file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
text: "public Job job"
description: "Open job()"
```

```java
@Bean
public Job job(JobRepository jobRepository, Step step1) {
	return new JobBuilder("BillingJob", jobRepository)
			.start(step1)
			.build();
}
```

You should also add the following import statement:

```java
import org.springframework.batch.core.job.builder.JobBuilder;
```

In this snippet, we replaced the creation of a `BillingJob` instance with the usage of the `JobBuilder` API to create the job.

We pass the job name and a reference to a `JobRepository`.

After that, we call the `JobBuilder.start` method that creates a sequential job flow and expects the first step of the sequence.

In our case, the first step is the `filePreparation` step which we defined as `step1` in the previous section.

### Remove `BillingJob`

Now that we have created the job as a sequence of steps, we can get rid of the initial implementation, the `BillingJob` class.

Go ahead and remove the `src/main/java/example/billingjob/BillingJob.java` file.

It is now time to run the application and see the new job implementation in action!
