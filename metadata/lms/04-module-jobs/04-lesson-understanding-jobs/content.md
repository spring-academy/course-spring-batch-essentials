In the previous lesson, you learned that a `Job` is an entity that encapsulates an entire batch process that runs from start to finish without interaction or interruption. In this lesson, you'll learn how `Job`s are represented internally in Spring Batch, understand how they're launched, and understand how their execution metadata is persisted.

# What Is a Job?

A `Job` is an entity that encapsulates an entire batch process that runs from start to finish. It consists of a set of steps that run in a specific order. We'll cover steps in a future lesson. Here, we focus on what a `Job` is and how it is represented in Spring Batch.

A batch job in Spring Batch is represented by the `Job` interface provided by the `spring-batch-core` dependency:

```java
public interface Job {
    String getName();
    void execute(JobExecution execution);
}
```

At a fundamental level, the `Job` interface requires that implementations specify the `Job` name (the `getName()` method) and what the `Job` is supposed to do (the `execute` method).

The `execute` method gives a reference to a `JobExecution` object. The`JobExecution` represents the actual execution of the `Job` at runtime. It contains a number of runtime details, such as the start time, the end time, the execution status, and so on. This runtime information is stored by Spring Batch in a metadata repository, which we'll cover in the next section.

Note how the `execute` method isn't expected to throw any exception. Runtime exceptions should be handled by implementations, and added in the `JobExecution` object. Clients should inspect the `JobExecution` status to determine success or failure.

# Understanding Job Metadata

One of the key concepts in Spring Batch is the `JobRepository`. The `JobRepository` is where all metadata about jobs and steps is stored. A `JobRepository` could be a persistent store, or an in-memory store. A persistent store has the advantage of providing metadata even after a `Job` is finished, which could be used for post analysis or to restart a `Job` in the case of a failure. We'll cover `Job` restartability in a later lesson.

Spring Batch provides a JDBC implementation of the `JobRepository`, which stores batch metadata in a relational database. In a production-grade system, you need to create a few tables that Spring Batch uses to store its execution metadata. We've covered metadata tables in the previous Lab.

The `JobRepository` is what creates a `JobExecution` object when a `Job` is first launched. But, how are `Job`s launched? Let's look at that in the next section.

# Launching Jobs

Launching jobs in Spring Batch is done through the `JobLauncher` concept, which is represented by the following interface:

```java
public interface JobLauncher {

   JobExecution run(Job job, JobParameters jobParameters)
          throws
             JobExecutionAlreadyRunningException,
             JobRestartException,
             JobInstanceAlreadyCompleteException,
             JobParametersInvalidException;
}
```

The `run` method is designed to launch a given `Job` with a set of `JobParameters`. We'll cover job parameters in detail in a later lesson. For now, you can think of them as a collection of key/value pairs that are passed to the `Job` at runtime. There are two important aspects to understand here:

- It is expected that implementations of the `JobLauncher` interface obtain a valid `JobExecution` from the `JobRepository` and execute the `Job`.
- The run method throws different types of exceptions. We'll cover all of these exceptions in detail during the course.

You'll almost never have to implement the `JobLauncher` interface yourself, because Spring Batch provides an implementation that's ready to use. The following diagram shows how the `JobLauncher`, the `JobRepository` and the `Job` interact with each other.

![Job-Launcher-Repository](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/job-launcher-repository.svg)

Batch jobs are typically launched in one of two ways:

- From the command line interface
- From within a web container

In this course, we'll only cover launching jobs from the command line. Please refer to the further resources links for more details about how to launch jobs from within a web container.
