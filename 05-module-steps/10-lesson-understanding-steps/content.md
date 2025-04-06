In the previous lessons, we discussed how to create a job, test it, and run it. We learned how to implement the `Job` interface and, specifically, implement the `execute` method to define what the job should do. As we said, you'll rarely have to implement the `Job` interface as such, since Spring Batch provides ready-to-use classes that let you define your job as a flow of steps. In this lesson, you'll learn about the different types of steps, what comprises them, and how to create a flow of steps to define the logic of your job.

## What Is a Step?

In everyday life we often talk about taking steps. A step down the path. A step in the right direction. Stepping up to the task.

Spring Batch has steps, too. A `Step` is a domain object that encapsulates an independent, sequential phase of a batch `Job`. It contains all of the information necessary to define a unit of work in a batch `Job`.

A `Step` in Spring Batch is represented by the `Step` interface provided by the `spring-batch-core` dependency:

```java
public interface Step {

  String getName();

  void execute(StepExecution stepExecution) throws JobInterruptedException;
}
```

Similar to the `Job` interface, the `Step` interface requires, at a fundamental level, an implementation to specify the step name (the `getName()` method) and what the step is supposed to do (the `execute` method).

The `execute` method provides a reference to a `StepExecution` object. The `StepExecution` represents the actual execution of the step at runtime. It contains a number of runtime details, such as the start time, the end time, the execution status, and so on. This runtime information is stored by Spring Batch in the metadata repository, similar to the `JobExecution`, as we have seen previously.

The `execute` method is designed to throw a `JobInterruptedException` if the job should be interrupted at that particular step.

## What Are the Different Types of Steps?

While it's possible to implement the `Step` interface manually to define the logic of a step, Spring Batch provides different implementations for common use cases. All these implementations derive from the base `AbstractStep` class that provides the common requirements such as setting the start time, end time of a step, updating the exit status of the step, persisting the step's metadata in the job repository, etc.

The most commonly used `Step` types are the following:

- `TaskletStep`: Designed for simple tasks (like copying a file or creating an archive), or item-oriented tasks (like reading a file or a database table).
- `PartitionedStep`: Designed to process the input data set in partitions.
- `FlowStep`: Useful for logically grouping steps into flows.
- `JobStep`: Similar to a `FlowStep` but actually creates and launches a separate job execution for the steps in the specified flow. This is useful for creating a complex flow of jobs and sub-jobs.

The following diagram explains the hierarchy and relation between these different types of `Steps`.

![Steps and Tasklets](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/bcbed634eed342346d30d61c0cfd8a30a55036a3/courses/course-spring-batch-essentials/step-and-tasklet.svg "Steps and Tasklets")

In this course, we'll focus on the `TaskletStep` type.

## The `TaskletStep`

The [`TaskletStep`](https://docs.spring.io/spring-batch/docs/5.0.4/reference/html/step.html#taskletStep "Spring Batch Tasklet Step") is an implementation of the `Step` interface based on the concept of a `Tasklet`. A `Tasklet` represents a unit of work that the Step should do when invoked. The `Tasklet` interface is defined as follows:

```java
@FunctionalInterface
public interface Tasklet {

  @Nullable
  RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception;
}
```

The `execute` method of this functional interface is designed to contain one iteration of the business logic of a `TaskletStep`. There are a few key elements to understand:

- The return type of the `execute` method is of type `RepeatStatus`. This is an enumeration that's used to signal to the framework that work has been completed (`RepeatStatus.FINISHED`), or not completed yet (`RepeatStatus.CONTINUABLE`). In that latter case, the `TaskletStep` re-invokes that `Tasklet` again.
- Each iteration of the `Tasklet` is executed in the scope of a database transaction. This way, Spring Batch saves the work that has been done during the iteration in the persistent job repository. That way, the step can resume where it left off, in case of failure. For this reason, the `TaskletStep` requires a `PlatformTransactionManager` to manage the transaction of the `Tasklet`. We'll address this in the Lab for this lesson.
- The `execute` method provides a reference to a `StepContribution` object, which represents the contribution of this `Tasklet` to the step (for example how many items were read, written, or otherwise processed) and a reference to a `ChunkContext` object, which is a bag of key/value pairs that provide detail about the execution context of the `Tasklet`.
- The `execute` method is designed to throw an exception if any error occurs during the processing, in which case, the step will be marked as failed.

Spring Batch provides several implementations of the `Tasklet` interface for common use cases:

- `ChunkOrientedTasklet`: Designed for item-oriented data sets, like a flat file or database table. We'll explain this implementation in more detail, and use it in the Labs of this course.
- `SystemCommandTasklet`: Lets you invoke an Operating System command within the `Tasklet`. We won't use this implementation in this course, but you can refer to the "References" section for more information about how to use it.
- Others. See the [Spring Batch Reference documentation](https://docs.spring.io/spring-batch/reference/index.html) for details.

We'll explore these concepts in more detail in future lessons. All you need to understand for now is the different types of Steps and what comprises them.
