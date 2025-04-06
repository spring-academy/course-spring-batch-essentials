As we mentioned, the path to the file in the skip listener will be configured with a job parameter. Therefore, we will declare the skip listener as a step-scope bean and use a SpEL expression to bind the value of the job parameter.

Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

```editor:open-file
file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
description: "Open BillingJobConfiguration.java"
```

```java
@Bean
@StepScope
public BillingDataSkipListener skipListener(@Value("#{jobParameters['skip.file']}") String skippedFile) {
	return new BillingDataSkipListener(skippedFile);
}
```

This method defines the `BillingDataSkipListener` as a step-scoped bean and configures it with a path to a file specified by the `skip.file` job parameter. This job parameter will be passed later when we launch the application.

Now that we defined the skip listener, let's move on to the skip policy.
