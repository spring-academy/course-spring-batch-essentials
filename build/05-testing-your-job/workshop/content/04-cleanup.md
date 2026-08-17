As we just saw, we encounter the "Job Instance already exists and is complete" error if we run our tests more than once.

Let's fix this using the `JobRepositoryTestUtils` class.

1. Clean up the metadata before tests run.

   Update the test class and add the following `@BeforeEach` initialization method:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "@SpringBootTest"
   description: Open BillingJobApplicationTests.java"
   ```

   ```java
   @BeforeEach
   public void setUp() {
      this.jobRepositoryTestUtils.removeJobExecutions();
   }
   ```

   You also need to add the following import statement:

   ```java
   import org.junit.jupiter.api.BeforeEach;
   ```

   This method uses the `JobRepositoryTestUtils` to clear all job executions before each test runs, so every run will have a fresh schema and won't be impacted by the metadata of other tests.

1. Rerun the test multiple times.

   Now if you run the test several times, the test should pass successfully without the "Job Instance already exists and is complete" error. What a relief!

Next, let's look at an alternative means to allow multiple executions of our test.
