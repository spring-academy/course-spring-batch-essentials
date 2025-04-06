In the previous Lab, you implemented the third and final step of the `BillingJob`. Billing data is now read from the input file, saved in the database and the billing report is generated in the staging directory.

There's still an issue though: The path to the input file is hard-coded in the bean definition method of `billingDataFileReader`:

```java
@Bean
public FlatFileItemReader<BillingData> billingDataFileReader() {
   return new FlatFileItemReaderBuilder<BillingData>()
	.name("billingDataFileReader")
	.resource(new FileSystemResource("staging/billing-2023-01.csv")) //hardcoded value
	.delimited()
	.names("dataYear", "dataMonth", "accountId", "phoneNumber", "dataUsage", "callDuration", "smsCount")
	.targetType(BillingData.class)
	.build();
}
```

What if we decide to ingest another file – say, `billing-2023-02.csv`? As currently defined, the same file (`billing-2023-01.csv`) will always be used, no matter which input file we pass as a job parameter. Since the input file is passed to the job as a parameter _at runtime_, there's no way to know its value at _configuration time_.

This is a common situation faced by many developers of batch applications: how to configure the item reader _lazily at runtime_ until the value of the job parameter is resolved?

This is where Spring Batch custom scopes come into play, and this is what you'll learn about in this lesson: job-scoped and step-scoped components.

## Understanding Batch Scopes

Spring Batch provides two custom Spring bean scopes: job scope and step scope. Batch-scoped beans are not created at application startup like singleton beans. Rather, they are created at runtime when the job or step is executed. A job-scoped bean will only be instantiated when the job is started. Similarly, a step-scoped bean won't be instantiated until the step is started. Since this type of bean is lazily instantiated at runtime, any runtime job parameter or execution context attribute can be resolved.

Configuring batch-scoped components requires a powerful Spring technology that we haven't introduced yet: the Spring Expression Language, or SpEL. Let's learn about it now.

## Spring Expression Language (SpEL)

Job parameters and execution context values can be resolved by using the Spring Expression Language, or SpEL. This powerful notation lets you specify which job parameter or context value should be bound in the bean definition method "late" at runtime. We talk about "Late binding" of job or step attributes. Let's see a simple example.

## Example of a Step-Scoped Component

The following example shows a step-scoped item reader bean:

```java
@Bean
@StepScope
public FlatFileItemReader<BillingData> reader(@Value("#{jobParameters['input.file']}") String inputFile) {
   return new FlatFileItemReaderBuilder<BillingData>()
   .resource(new FileSystemResource(inputFile))
	   // other properties

	   .build();
}
```

This snippet creates a step-scoped bean of type `FlatFileItemReader`, thanks to the `@StepScope` annotation. This annotation is provided by Spring Batch and marks a bean as step-scoped. A similar annotation (`@JobScope`) exists for job-scoped beans as well.

This item reader is configured with an input file from a job parameter called `input.file`. The job parameter is specified by using the `@Value("#{jobParameters['input.file']}")` notation, which instructs Spring Framework to resolve the value `lazily` from the `jobParameters` object. With that in place, this item reader will be configured dynamically with the value of the input file resolved at runtime from job parameters. We'll apply this concept where appropriate to the item readers and writers of our `BillingJob` in the lab for this lesson.
