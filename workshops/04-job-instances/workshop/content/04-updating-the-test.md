In the previous section, you have seen how to pass `JobParameter`s declaratively on the command line interface with key/value pairs like `input.file=src/main/resources/billing-2023-02.csv`.
In this section, we will update the test of our `BillingJob` to show you how to pass `JobParameter`s programmatically through the APIs provided by Spring Batch.

Compared to the previous version of our `BillingJob`, in this Lab we have updated the logic of our `Job` to extract the input file from the `JobParameter`s set and print a message to the console accordingly.
We should update the test logic accordingly.

1. Update the `BillingJobApplicationTests` to test the new logic.

   In the `Editor` tab, go to the `src/test/java/example/billingjob/BillingJobApplicationTests.java` and update the test method `testJobExecution` with the following code:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "void testJobExecution"
   description: "Open testJobExecution()"
   ```

   ```java
   @Test
   void testJobExecution(CapturedOutput output) throws Exception {
       // given
       JobParameters jobParameters = new JobParametersBuilder()
               .addString("input.file", "/some/input/file")
               .toJobParameters();
       // when
       JobExecution jobExecution = this.jobLauncher.run(this.job, jobParameters);
       // then
       Assertions.assertTrue(output.getOut().contains("processing billing information from file /some/input/file"));
       Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
   }
   ```

   You should also add the following import statement:

   ```java
   import org.springframework.batch.core.JobParametersBuilder;
   ```

   The `JobParametersBuilder` is the main API provided by Spring Batch to build a set of `JobParameter`s. In this test, we use that builder to create a parameter of type `String` named `input.file` having the `/some/input/file` value.

   The value of the parameter does not really matter for the purpose of the test, all we have to verify is that the `Job` is receiving the right parameter value and printing the message as expected. This is what we have done in the following assertion:

   ```java
   Assertions.assertTrue(output.getOut().contains("processing billing information from file /some/input/file"));
   ```

2. Run the test.

   To execute the test, open the **Terminal** tab and run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw test
   ```

   The test should pass, which means our batch `Job` is doing what it is supposed to do.

   ### Learning Moment: Run the tests again

   If you run the test a second time, it will fail with a similar error that we saw during the lesson when running the same completed `JobInstance` again:

   ```shell
   org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException:
    A job instance already exists and is complete for parameters={'input.file':'{value=/some/input/file, type=class java.lang.String, identifying=true}'}.
    If you want to run this job again, change the parameters.
   ```

   This is because the same database is used for all test runs. While this is intended in production, it could be problematic in tests as project builds won't be idempotent.

   For this reason, Spring Batch provides a utility class called `JobRepositoryTestUtils` that allows you to clean-up the database before or after each test.
   This is what we will discuss in the next Lesson.

   For now you will need to reset the database if you encounter this error.

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ scripts/drop-create-database.sh
   ```

3. Going further with `JobParamter`s.

   In addition to the ability of providing the name, type and value of `JobParameter`s, the `JobParametersBuilder` API also allows you to specify if the parameter is identifying or not through the third boolean parameter of the `addString` method. Here is an example:

   ```java
    JobParameters jobParameters = new JobParametersBuilder()
                                     .addString("input.file", "/some/input/file")
                                     .addString("file.format", "csv", false)
                                     .toJobParameters();
   ```

   With this snippet, we add a second parameter named `file.format` of type `String` with the value `csv` and which is **not** identifying (due to the third `false` method parameter).

   Note how we did not explicitly pass `true` as third parameter for `input.file` as this is the default value for `JobParameter`s in Spring Batch.

   As a further exercise, you can try to launch the `BillingJob` with a mix of identifying and non-identifying `JobParameter`s and see how this impacts or not the definition and identification of `JobInstance`s.
