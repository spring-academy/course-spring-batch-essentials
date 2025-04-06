In the previous lesson, you learned about `Job`s and `JobExecution`s. In this lesson, we'll explore another key concept from the Batch domain model, which is `JobInstance`. We'll explain what `JobInstance`s are, and how they relate to `Job`s and `JobExecution`s.

## What Are Job Instances?

A `Job` might be defined once, but it'll likely run many times, commonly on a set schedule. In Spring Batch, a `Job` is the generic definition of a batch process specified by a developer. This generic definition must be parametrized to create actual instances of a `Job`, which are called `JobInstance`s.

A `JobInstance` is a unique parametrization of a `Job` definition. For example, imagine a batch process that needs to be executed once at the end of each day, or when a certain file is present. In the once-per-day scenario, we can use Spring Batch to create an `EndOfDay` `Job` for that. There would be a single `EndOfDay` `Job` definition, but multiple instances of that same `Job`, one per day. Each instance would process the data of a particular day, and might have a different outcome (success or failure) from other instances. Therefore, each individual instance of the `Job` must be tracked separately.

A `JobInstance` is distinct from other `JobInstance`s by a specific parameter, or a set of parameters. For example, a parameter named `schedule.date` would specify a specific day. Such a parameter is called a `JobParameter`. `JobParameter`s are what distinguish one `JobInstance` from another. The following diagram shows how `JobParameter`s define `JobInstance`s:

![Job Parameters](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/job-parameters.svg)

## What Do Job Instances and Job Parameters Represent?

`JobInstance`s are distinct from each other by distinct `JobParameter`s. Those parameters usually represent the data intended to be processed by a given `JobInstance`. For example, in the case of the `EndOfDay` `Job`, the `schedule.date` `JobParameter` for January 1st defines the `JobInstance` that will process the data of January 1st. The `schedule.date` `JobParameter` for January 2nd defines the `JobInstance` that will process the data of January 2nd, and so forth.

While it is not required for `Job` parameters to represent the data to be processed, this is a good hint - and a good practice - to correctly design `JobInstance`s. Designing `JobInstance`s to represent the data to be processed is easier to configure, to test, and to think about, in case of failure.

The definition of a `JobInstance` itself has absolutely no bearing on the data to be loaded. It is entirely up to the `Job` implementation to determine how data is loaded, based on `JobParameter`s. Here are a few examples of `JobParameter`s, and how they represent the data to be processed by the corresponding `JobInstance`:

- A specific date: In this case, we would have a `JobInstance` per date.
- A specific file: In this case, we would have a `JobInstance` per file.
- A specific range of records in a relational database table: In this case, we would have a `JobInstance` per range.
- And more.

For our course, the `BillingJob` for Spring Cellular consumes a flat file as input, which is a good candidate to be passed as a `JobParameter` to our `Job`. This is what we'll see in the upcoming Lab of this lesson.

## How Do Job Instances Relate to Job Executions?

A `JobExecution` refers to the technical concept of a single _attempt_ to run a `JobInstance`. As seen in the previous lesson, a `JobExecution` may end in a success or failure. In the case of the `EndOfDay` `Job`, if the January 1st run fails the first time and is run again the next day, it is still the January 1st run. Therefore, each `JobInstance` can have multiple `JobExecution`s.

The relation between the concepts of `Job`, `JobInstance`, `JobParameters`, and `JobExecution` is summarized in the following diagram:

![Relationships between various Job classes](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/job-class-relations.svg)

Here's a concrete example of the lifecycle of a `JobInstance` in the case of the `EndOfDay` `Job`:

![Example of Job Instance lifecycle using the end-of-day scenario](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/lifecycle-example.svg)

In this example, the first execution attempt of `Job Instance 1` fails, so another execution is run and succeeds. This leads to two `JobExecution`s for the same `JobInstance`. For `Job Instance 2` however, the first execution attempt succeeds, therefore there is no need to launch a second execution.

In Spring Batch, a `JobInstance` is not considered to be complete unless a `JobExecution` completes successfully. A `JobInstance` that is complete can't be restarted again. This is a design choice to prevent accidental re-processing of the same data for batch `Job`s that are not idempotent.

## The Different Types of Job Parameters

`JobParameter`s are typically used to distinguish one `JobInstance` from another. In other words, they are used to _identify_ a specific `JobInstance`.

Not all parameters can be used to identify `Job` instances. For example, if the `EndOfDay` `Job` takes another parameter - say, `file.format`) - that represents the format of the output file (CSV, XML, and others), this parameter does not really represent the data to process, so, it could be excluded from the process of identifying the `Job` instances.

This is where non-identifying `JobParameter`s come into play. In Spring Batch, `JobParameter`s can be either identifying or non-identifying. An identifying `JobParameter` contributes to the identification of `JobInstance`, while a non-identifying one doesn't. By default, `JobParameter`s are identifying, and Spring Batch provides APIs to specify whether a `JobParameter` is identifying or not.

In the example of the `EndOfDay` `Job`, the parameters can be defined in the following table:

| Job parameter | Identifying? | Example    |
| ------------- | ------------ | ---------- |
| schedule.date | Yes          | 2023-01-01 |
| file.format   | No           | csv        |

Now the question is: Why is this important, and how is it used in Spring Batch? Identifying `JobParameter`s play a crucial role in the case of failure. In a production environment, where hundreds of `Job` instances are running, and one of them fails, we need a way to identify which instance has failed. This is where identifying `Job` parameters are key. When a `JobExecution` for a given `JobInstance` fails, launching the same job with the _same_ set of identifying `JobParameter`s will create a new `JobExecution` (ie a new attempt) for the _same_ `JobInstance`.

You'll practice several concrete examples of this concept in the accompanying hands-on Lab of this lesson.
