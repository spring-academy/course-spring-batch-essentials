Now that our job implementation is complete, we can write a test for it.
Spring Batch provides several test utilities to simplify testing Batch components. We cover those later in the course.

For this lesson, we see how to test a Spring Batch `Job` by using JUnit 6 and the test utilities provided by Spring Boot.

1. Update the `BillingJobApplicationTests`.

   In the `Editor` tab, go to the `src/test/java/example/billingjob/BillingJobApplicationTests.java` file and update its content with the following code:

   ```editor:open-file
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   description: "Open BillingJobApplicationTests.java"
   ```

   ```java
   package example.billingjob;

   import org.junit.jupiter.api.Assertions;
   import org.junit.jupiter.api.Test;
   import org.junit.jupiter.api.extension.ExtendWith;

   import org.springframework.batch.core.ExitStatus;
   import org.springframework.batch.core.job.Job;
   import org.springframework.batch.core.job.JobExecution;
   import org.springframework.batch.core.job.parameters.JobParameters;
   import org.springframework.batch.core.launch.JobOperator;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.system.CapturedOutput;
   import org.springframework.boot.test.system.OutputCaptureExtension;
   import org.springframework.test.context.TestPropertySource;

   @SpringBootTest
   @ExtendWith(OutputCaptureExtension.class)
   @TestPropertySource(properties = "spring.batch.job.enabled=false")
   class BillingJobApplicationTests {

   	@Autowired
   	private Job job;

   	@Autowired
   	private JobOperator jobOperator;

   	@Test
   	void testJobExecution(CapturedOutput output) throws Exception {
   		// given
   		JobParameters jobParameters = new JobParameters();

   		// when
   		JobExecution jobExecution = this.jobOperator.run(this.job, jobParameters);

   		// then
   		Assertions.assertTrue(output.getOut().contains("processing billing information"));
   		Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
   	}
   }
   ```

2. Understanding the test class structure

   - First, we annotate the test class with `@SpringBootTest`. Doing so enables testing features from Spring Boot, which comprises loading the Spring application context, preparing the test context, and so on.

     ```java
     @SpringBootTest
     class BillingJobApplicationTests {
     ...
     }
     ```

   - After that, and for the purpose of testing our batch `Job` that writes output to the console, we use the `OutputCaptureExtension` that is provided by Spring Boot to capture any output that is written to the standard output and the error output.

     In our case, we need this output capture to check if the job is correctly printing the `processing billing information` message to the console.

     ```java
     @SpringBootTest
     @ExtendWith(OutputCaptureExtension.class)
     class BillingJobApplicationTests {
     ...
     }
     ```

   - Finally we add `@TestPropertySource(properties = "spring.batch.job.enabled=false")` to disable the automatic execution of Spring Batch jobs when the test application context boots up. This prevents the automatic creation of a job instance before the test is launched.
  
      ```java
      @SpringBootTest
      @ExtendWith(OutputCaptureExtension.class)
      @TestPropertySource(properties = "spring.batch.job.enabled=false")
      class BillingJobApplicationTests {
      ...
      }
     ```

   - We can then autowire the `Job` under test as well as `JobOperator` from the test context.

     ```java
     ...
     @Autowired
     private Job job;

     @Autowired
     private JobOperator jobOperator;
     ...
     ```

3. Understanding the test case

   Now that the test class is configured, we can write the test method `testJobExecution`, which is designed to:

   - Launch the batch `Job` with a set of parameters.

     The parameters are empty for now, but that is fine for the purpose of this lab. We will use `Job` parameters later in the course.

     ```java
     @Test
     void testJobExecution(CapturedOutput output) throws Exception {
     	// given
     	JobParameters jobParameters = new JobParameters();

     	// when
     	JobExecution jobExecution = this.jobOperator.run(this.job, jobParameters);

     	...
     }
     ```

   - Check that the output of the job contains the expected message and that its status has been updated correctly.

     ```java
     @Test
     void testJobExecution(CapturedOutput output) throws Exception {
     	...

     	// then
     	Assertions.assertTrue(output.getOut().contains("processing billing information"));
     	Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
     }
     ```

   That's all we need to test our billing `Job`.

4. The test will attempt to create a new instance of the job and will fail as an instance already exists from previous runs. Clean up the previously run `Job`. In the **Terminal** tab, run the following commands:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ scripts/drop-create-database.sh
   ```

5. Run the test.

   To execute the test from the `Editor` tab, right-click on the `src/test/java/example/billingjob/BillingJobApplicationTests.java` and select `Run Java`.

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "@SpringBootTest"
   description: "Right-click ➡️ Run Java"
   ```

   The test should pass, which means our batch `Job` is doing what it is supposed to do.
