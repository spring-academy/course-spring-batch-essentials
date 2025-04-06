We've already seen how to test Spring Batch jobs using JUnit 5 and Spring Boot test utilities in previous Labs. In this lesson, we'll focus on the test utilities provided by Spring Batch in the `spring-batch-test` module, which is designed to simplify testing batch artifacts.

## The Different Types of Tests for Batch Jobs

When it comes to testing batch jobs, there are several levels of testing:

- **Testing the job from end to end:** In this scenario, a test should provide the input data, execute the job, and verify the end result. We can qualify this kind of testing as "black-box" testing, where we consider the job as a black box that we test based on inputs and outputs. End to end testing is what we have been doing thus far in this course.

- **Testing each step of the job individually:** In this scenario, a complex batch job is defined in a workflow of steps, and we test each step in isolation without launching the entire job.

In both cases, there's a need to set up test data and launch a job, or a specific step. For this requirement, Spring Batch provides the `JobLauncherTestUtils` API that is designed to launch entire jobs or individual steps in tests. The `JobLauncherTestUtils` provides several utilities and methods. Here are the most important ones:

- **Random job parameters generation:** this feature allows you to generate a unique set of job parameters in order to have distinct job instances during tests. This is particularly useful to make your tests repeatable, and your builds idempotent. In fact, this prevents restarting the same job instances again across different tests, which would make some tests fail. These methods include:

  - `JobLauncherTestUtils.getUniqueJobParameters`
  - `JobLauncherTestUtils.getUniqueJobParametersBuilder`

- **Launching an entire job from end to end:** `JobLauncherTestUtils.launchJob` allows you to launch a job in the same way you would launch it in production. You have the choice to launch it with a set of randomly generated job parameters, or with a specific set of parameters.

- **Launching an individual step:** `JobLauncherTestUtils.launchStep` allows you to test a step in isolation from other steps without having to launch the enclosing job.

We'll use these utilities in the upcoming Lab for this lesson.

## Database Decisions

When testing Spring Batch jobs, it is important to decide how to manage metadata across tests to avoid restartability issues between job instances. Nobody wants to see their tests failing with that familiar "A job instance already exists and is complete" error!

The most typical choice for keeping test runs idempotent is using a disposable, in-memory database for each test like H2, HSQL, or Derby. Although using an in-memory database for tests is a good option to avoid metadata sharing, it has the disadvantage of often being different from the database used in production. This might be an unacceptable risk for teams that need to minimize the number of differences between the different parts of their system, including differences between test and live infrastructure.

For this reason, we'll demonstrate running tests against an instance of a production-grade database: the very popular PostgreSQL database.

## Database Sharing Trade-Offs

So, we've decided to use a "real" database when running our tests. Now, we face another decision: how will we install and manage this database?

A database instance can be installed as a separate process on the test machine, or run in a containerized environment like Docker. When using a containerized environment, libraries like TestContainers can be helpful to create disposable database containers for tests. You can learn more about TestContainers using the resource links provided in this lesson.

Now, the question is: should the same database instance (be it run as a separate process, or in a container) be shared between all tests or should each test have its own database instance?

While making each test have its own database instance is technically possible, it has the drawback of being expensive, both in terms of execution time, and in resources. In fact, we'll need to create and destroy real database instances or containers for each test, and execute the Spring Batch metadata initialization scripts, every time. Alternatively, we could try to set up fancy (and error-prone) configuration with database transactions or rollbacks. This might work with a few tests, but in our experience these techniques tend to be unbearably slow if scaled to hundreds (or even thousands) of tests, which is typically the case in real projects.

For the above reasons we have chosen the following option for this course: use a shared instance of the same database product between tests. But, in this scenario, we'll need to make sure that metadata is cleared between tests, in order to avoid the dreaded "A job instance already exists and is complete" error.

## Spring Batch Testing Utilities

To help with database cleanup during tests, Spring Batch provides the `JobRepositoryTestUtils` that can be used to create or delete `JobExecution`s as needed during tests.

A typical usage of `JobRepositoryTestUtils` is to clear the batch metadata from the database **before** or **after** each test run. This allows you to have a clean environment for each test case without compromising the test suite execution time, or performance. For example:

```java
@BeforeEach
void setUp() {
    this.jobRepositoryTestUtils.removeJobExecutions();
}
```

Or

```java
@AfterEach
void tearDown() {
    this.jobRepositoryTestUtils.removeJobExecutions();
}
```

Which option you choose might depend on how you set up or tear down your tests.

## Additional Test Utilities

You might need to do more in your tests than clean up the database. Luckily, `JobLauncherTestUtils` and `JobRepositoryTestUtils` are not the only test utilities that Spring Batch provides in the `spring-batch-test` module.

Other utilities include, but not limited to, the following items:

- **The `ExecutionContextTestUtils` class:** this class provides static methods to access attributes from the execution context of `JobExecution`s and `StepExecution`s.
- **The `MetaDataInstanceFactory` class:** this class is helpful to create metadata entities, such as `JobInstance` and `JobExecution`, with the constraints defined by the batch domain model, such as the parent-child link between `JobInstance` and `JobExecution`, or the parent-child link between `JobExecution` and `StepExecution`.
- **The `@SpringBatchTest` annotation:** This annotation registers test utilities (like `JobLauncherTestUtils`, `JobRepositoryTestUtils`, etc) as beans in the test context in order to be able to use them in tests.

### Preview: Spring Batch Scopes Test Utilities

In addition, you might need fine-grained management of your Batch artifacts during tests. Spring Batch provides additional utilities, such as test execution listeners like `JobScopeTestExecutionListener` and `StepScopeTestExecutionListener`. These listeners are used to test components (like item readers, item writers, etc) that are scoped with the custom Spring scopes provided by Spring Batch which are `JobScope` and `StepScope`.

We haven't addressed scoped beans yet in the course, but we'll address them in a future module. For now, you just need to know that those test listeners are part of the utilities provided by Spring Batch in the `spring-batch-test` module, and we'll use them in future Labs. Stay tuned!
