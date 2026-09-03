In the previous lesson we learned all about _what_ Spring Batch Steps are. Now let's learn how to use them in our applications.

## Using Steps To Define the Job Execution Flow

As we said, you'll rarely have to implement the `Job` interface manually, like we did in the previous lessons (which we did deliberately, for learning purposes). In fact, Spring Batch provides the `AbstractJob` class that lets you define your job as a flow of steps. This class has two variations:

- `SimpleJob`: For sequential execution of steps.
- `FlowJob`: For complex step flows, including conditional branching and parallel execution.

In this course, you'll learn how to use the `SimpleJob` variation. For more information about `FlowJob`, see the "Condition Flows" and "Parallel Flows" references in the "Links" section of this lesson.

## The `SimpleJob`

The `SimpleJob` class is designed to compose a job as a sequence of steps. A step should be completed successfully in order for the next step in the sequence to start. If a step fails, the job is immediately terminated and subsequent steps are not executed. You can create a sequential flow of steps by using the `JobBuilder` API. Here's an example with two steps:

```java
@Bean
public Job myJob(JobRepository jobRepository, Step step1, Step step2) {
  return new JobBuilder("job", jobRepository)
    .start(step1)
    .next(step2)
    .build();
}
```

In this example, the job (`myJob`) is defined as a sequence of two steps, `step1` and `step2`. The job starts with `step1` and moves to `step2` only if `step1` completes successfully. If `step1` fails, the job is terminated, and `step2` won't be executed. Our `BillingJob` for Spring Cellular is defined as a sequence of three steps, and we'll define it in a similar manner in the Lab for this lesson.

In the previous example, we pass steps as parameters to the `myJob` bean definition method. But, how are these steps defined, and how are they created? This is what we'll see in the next section.

## How To Create Steps?

Similar to the `JobBuilder` API, Spring Batch provides the `StepBuilder` API to let you create different types of steps. All step types share some common properties (like the step name, the job repository to report metadata to, and others), but each step type has its own specific properties. For this reason, Spring Batch provides a specific builder for each step type (`TaskletStepBuilder`, `PartitionedStepBuilder`, and others).

You shouldn't worry about how to create those specific builders, as Spring Batch guides you to using the corresponding one depending on the type of step you are creating. The main entry point to create steps is the `StepBuilder` API. Here's an example that creates a `TaskletStep`:

```java
@Bean
public Step taskletStep(JobRepository jobRepository, Tasklet tasklet, PlatformTransactionManager transactionManager) {
  return new StepBuilder("step1", jobRepository)
    .tasklet(tasklet, transactionManager)
    .build();
}
```

In this example, we define the step as a Spring bean. The `StepBuilder` accepts the step name and the job repository to report metadata to at construction time, as those are common to all step types.

After that, we call the `StepBuilder.tasklet` method, which will use a `TaskletStepBuilder` to further define specific properties of the `TaskletStep`, mainly the `Tasklet` to execute as part of the step and the transaction manager to use for transactions. Note how we did not use the `TaskletStepBuilder` directly.

The pattern is similar if, for instance, you want to create a partitioned step. Here's an example to create a `PartitionedStep`:

```java
@Bean
public Step partitionedtStep(JobRepository jobRepository, Partitioner partitioner) {
  return new StepBuilder("step1", jobRepository)
    .partitioner("worker", partitioner)
    .build();
}
```

In the same way that we've created a `TaskletStep` in our lab, we'll use the main entry point (the `StepBuilder`) by passing the common properties to it in the constructor (that is, the step name and job repository), then call `StepBulider.partitioner` to further configure the partitioned step with its specific attributes (the `partitioner` in this case). The `Partitioner` is beyond the scope of this lesson and course. We'll use it here only as an example, to show the difference between step type specific attributes. Note that we didn't use the `PartitionedStepBuilder`, instead we used the `StepBuilder` directly.

## Understanding Step Metadata

Similar to job-level metadata, which is stored in the `BATCH_JOB_EXECUTION` table, Spring Batch stores step-level metadata in the `BATCH_STEP_EXECUTION` table. This includes the start time of the step, its end time, its execution status, and other details. We'll see an example of this step-level metadata in the lab for this course.

Another similarity with the job level is the execution context. Each step has an execution context, which is nothing more than a set of key/value pairs to store runtime information about the execution of the step. This includes the step type, the tasklet type if the step is a `TaskletStep`, and other details. The context might also record the progress of a step, like item read count, item write count, and other metrics. These key/value pairs can be used to restart a step where it left off, in case of failure.

By default, a successful step is not re-executed when restarting a failed job instance. However, in some situations, even a successful step should be re-executed when re-attempting a job instance. Spring Batch makes that possible through the `StepBuilder.allowStartIfComplete` parameter. You can also limit the number of times a step is restarted by using the `StepBuilder.startLimit` parameter.
