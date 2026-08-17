In the accompanying lesson we learned that `JobLauncherTestUtils` provides helper methods that create random job parameters to help us avoid the dreaded "Job Instance already exists and is complete" error.

Let's explore this utility now.

1. Utilize unique job parameters.

   Update our test method and change the way we build `JobParameters`.

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "void testJobExecution"
   description: "Open testJobExecution()"
   ```

   ```java
   @Test
   void testJobExecution(CapturedOutput output) throws Exception {
      // given
      JobParameters jobParameters = this.jobLauncherTestUtils.getUniqueJobParametersBuilder()
         .addString("input.file", "/some/input/file")
         .toJobParameters();
   ```

   Note that we only changed `new JobParametersBuilder()` to `this.jobLauncherTestUtils.getUniqueJobParametersBuilder()`.

1. Run the test.

   When you run the test you'll see it passes just as it had in the last Lab step.

   But what would happen if we did not clean up the metadata in the database?

1. Disable `removeJobExecutions()`.

   For now, comment-out the line that cleaned up the database:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "@BeforeEach"
   description: "Open @BeforeEach"
   ```

   ```java
   @BeforeEach
   public void setUp() {
   //  *** Comment out the line below **
   //  this.jobRepositoryTestUtils.removeJobExecutions();
   }
   ```

   This was very problematic in the past. What will happen now when we rerun the tests?

1. Rerun the tests.

   Rerun the test and notice that it passes!

   In fact, rerun the test multiple times: 2 times, 10 times -- as many times as you'd like. The test keeps passing and does not fail with "Job Instance already exists and is complete".

   But how is this possible?

   Let's look at the metadata to find out.

1. Inspect the database.

   We've seen in previous labs that jobs save metadata to the database.

   Let's look at some of that metadata now that we are using `jobLauncherTestUtils.getUniqueJobParametersBuilder()` but _not_ `jobRepositoryTestUtils.removeJobExecutions();`

   Run the following query in the **Terminal**:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION_PARAMS;'
   ```

   Your output should look similar to the following:

   ```shell
       job_execution_id | parameter_name |  parameter_type  |   parameter_value    | identifying
   ------------------+----------------+------------------+----------------------+-------------
                  49 | random         | java.lang.Long   | -2305569543280804739 | Y
                  49 | input.file     | java.lang.String | /some/input/file     | Y
                  51 | random         | java.lang.Long   | 208082855255757609   | Y
                  51 | input.file     | java.lang.String | /some/input/file     | Y
                  53 | random         | java.lang.Long   | 6393756363010672354  | Y
                  53 | input.file     | java.lang.String | /some/input/file     | Y
   ```

   Look at all that metadata!

   Note that the automatic inclusion of the `random` parameter, which is keeping our `JobExecution`s unique.

   It's clear that more and more metadata will be added to the database over time. Should we leave this metadata in the database since it does not seem to impact our test runs? Is this a "who cares?" situation?

   ### Learning moment: test pollution

   Yes, we should reinstate `jobRepositoryTestUtils.removeJobExecutions()` and clean the database before each test run. But why does it matter if the tests pass anyway?

   Even though _these particular tests currently pass_ without cleaning up the database, we have still introduced _unmitigated test pollution_ into the equation. What is test pollution and why is it a problem?

   Test pollution occurs when tests leave artifacts behind that might impact other or future tests. By omitting `removeJobExecutions()`, tests might need to account for the metadata left by other tests. Imagine hundreds or even thousands of tests that are all required to somehow account for the leftover artifacts of every other test!

   Our current test passes even with test pollution present, but other tests that might be written in the future might not be so lucky. Perhaps we need to count test runs, or verify exact metadata output, or any number of other scenarios that require precise control over our test output.

   For these and many other reasons, cleaning up any test artifacts before or after a test run is a best practice.

1. Reinstate `removeJobExecutions()`.

   Update the test to once again clean up before each test run:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "@BeforeEach"
   description: "Open @BeforeEach"
   ```

   ```java
   @BeforeEach
   public void setUp() {
      this.jobRepositoryTestUtils.removeJobExecutions();
   }
   ```

1. Rerun the tests and inspect the database.

   Rerun the tests several times and notice that the database only contains the data from the previous test run:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION_PARAMS;'

       job_execution_id | parameter_name |  parameter_type  |   parameter_value    | identifying
   ------------------+----------------+------------------+----------------------+-------------
                  68 | random         | java.lang.Long   | -2999569543280804739 | Y
                  68 | input.file     | java.lang.String | /some/input/file     | Y
   ```
