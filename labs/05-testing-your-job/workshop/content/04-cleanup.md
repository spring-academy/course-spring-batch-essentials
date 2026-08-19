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

1. Run the test.

   Right click on `src/test/java/example/billingjob/BillingJobApplicationTests.java` and select "Run Java".

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "@SpringBootTest"
   description: "Right-click ➡️ Run Java"
   ```

   What happens?

   The test fails. But why, if we added the `@BeforeEach` method? Remember that Spring Boot automatically launches the job with no parameters every time the application context starts, including for tests. If a job instance with no parameters had already completed *before* you added this `@BeforeEach` method as done in the previous steps the application context will fail to start before `@BeforeEach` gets a chance to run, since the cleanup itself depends on a context that hasn't loaded yet. Run `scripts/drop-create-database.sh` once to clear that leftover state; after that, the `@BeforeEach` cleanup keeps every subsequent run working on its own.

2. Rerun the test multiple times.

   Now if you run the test several times, the test should pass successfully without the "A job instance already exists and is complete" error. What a relief!

Next, let's look at an alternative means to allow multiple executions of our test.
