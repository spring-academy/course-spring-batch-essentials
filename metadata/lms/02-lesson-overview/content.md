In this lesson, we describe what batch processing is, and the challenges that come with it. We also introduce the Spring Batch framework, explain its domain model, and its internal architecture.

## Introduction to Batch Processing

Batch processing is a method of processing large volumes of data simultaneously, instead of processing them individually, in real time (in that case, we could talk about stream processing). This approach is widely used in many industries, including finance, manufacturing, and telecommunications. Batch processing is often used for tasks that require the processing of large amounts of data, such as payroll processing or billing, as well as tasks that require time-consuming calculations or analysis. Batch applications are ephemeral, which means that once they've completed, they end.

This type of processing comes with a number of challenges, including, but not limited to:

- Handling large amounts of data efficiently
- Tolerance to human errors and hardware deficiencies
- Scalability

When it's time to provide a batch-based application for processing large amounts of data in a structured way, Spring Batch provides a robust and efficient solution. So, what is Spring Batch exactly? How does it help address batch processing challenges? Let's find out!

## Spring Batch Framework

Spring Batch is a lightweight, comprehensive framework, designed to enable the development of robust batch applications that are vital for the daily operations of enterprise systems.

It provides all the necessary features that are essential for processing large volumes of data, including transaction management, job processing status, statistics, and fault-tolerance features. It also provides advanced scalability features that enable high-performance batch jobs through multi-threaded processing and data-partitioning techniques. You can use Spring Batch in both simple use cases (such as loading a file into a database), and complex, high-volume use cases (like moving data between databases, transforming it, and so on).

Spring Batch integrates seamlessly with other Spring technologies, making it an excellent choice for writing batch applications with Spring.

## Batch Domain Language

The key concepts of the Spring Batch domain model are represented in the following diagram:

![Batch domain Model](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/overview-lesson-domain-model.svg)

A `Job` is an entity that encapsulates an entire batch process, that runs from start to finish without interruption. A `Job` has one or more steps. A `Step` is a unit of work that can be a simple task (such as copying a file or creating an archive), or an item-oriented task (such as exporting records from a relational database table to a file), in which case, it would have an `ItemReader`, an `ItemProcessor` (which is optional), and an `ItemWriter`.

A `Job` needs to be launched with a `JobLauncher`, and can be launched with a set of `JobParameter`s. Execution metadata about the currently running `Job` is stored in a `JobRepository`.

We will cover each of these key concepts in detail throughout the course.

## Batch Domain Model

Spring Batch uses a robust and well-designed model for the batch processing domain. It provides a rich set of Java APIs with interfaces and classes that represent all of the key concepts of batch processing like `Job`, `Step`, `JobLauncher`, `JobRepository`, and more. We will use these APIs in this course.

While the batch domain model can be implemented with any persistence technology (like a relational database, a non-relational database, a graph database, etc), Spring Batch provides a relational model of the batch domain concepts with metadata tables that closely match the classes and interfaces in the Java API.

The following entity-relationship diagram presents the main metadata tables:

![Batch relational Model](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/overview-lesson-relational-model.svg)

- `Job_Instance`: This table contains all information relevant to a job definition, such as the job name and its identification key.
- `Job_Execution`: This table holds all information relevant to the execution of a job, like the start time, end time, and status. Every time a job is run, a new row is inserted in this table.
- `Job_Execution_Context`: This table holds the execution context of a job. An execution context is a set of key/value pairs of runtime information that typically represents the state that must be retrieved after a failure.
- `Step_Execution`: This table holds all information relevant to the execution of a step, such as the start time, end time, item read count, and item write count. Every time a step is run, a new row is inserted in this table.
- `Step_Execution_Context`: This table holds the execution context of a step. This is similar to the table that holds the execution context of a job, but instead it stores the execution context of a step.
- `Job_Execution_Params`: This table contains the runtime parameters of a job execution.

## Spring Batch Architecture

Spring Batch is designed in a modular, extensible way. The following diagram shows the layered architecture that supports the ease of use of the framework for end users:

![Batch relational Model](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/overview-lesson-architecture.svg)

This layered architecture highlights three major high-level components:

- The `Application` layer: contains the batch job and custom code written by the developers of the batch application.
- The `Batch Core` layer: contains the core runtime classes provided by Spring Batch that are necessary to create and control batch jobs. It includes implementations for `Job` and `Step`, as well as common services like `JobLauncher` and `JobRepository`.
- The `Batch Infrastructure` layer: contains common item readers and writers provided by Spring Batch, plus base services such as the repeat and retry mechanisms, which are used both by application developers and the core framework itself.

As a Spring Batch developer, you typically use APIs provided by Spring Batch in the `Batch Infrastructure` and `Batch Core` modules to define your jobs and steps in the `Application` layer. Spring Batch provides a rich library of batch components that you can use out of the box (such as item readers, item writers, data partitioners, and more).
