One of the features of `JobLauncherTestUtils` is that it automatically detects the job under test in the application context if it is unique. This is the case in our Lab, we only have a single job that is defined which is the `BillingJob`. For this reason, we can remove the autowiring of the job under test from the test class.

1. Use the utility to launch the job.

   Instead of autowiring the `Job` under test and the `JobLauncher` to test it, we use the `jobLauncherTestUtils.launchJob` method which has the same effect. Let's try that.

   Modify following lines in the test class:

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
       // ** Update the following line:
       JobExecution jobExecution = this.jobLauncherTestUtils.launchJob(jobParameters);

       // then
       Assertions.assertTrue(output.getOut().contains("processing billing information from file /some/input/file"));
       Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
   }
   ```

1. Remove the unused variables.

   Since we just removed the only usage of the `job` and `jobLauncher` we can delete their declarations, too.

   ```java
   @SpringBootTest
   @SpringBatchTest
   @ExtendWith(OutputCaptureExtension.class)
   class BillingJobApplicationTests {

   //  ** Remove these lines below:
   //  @Autowired
   //  private Job job;

   //  @Autowired
   //  private JobLauncher jobLauncher;
   ...
   }
   ```

1. Run the test.

   Right click on `src/test/java/example/billingjob/BillingJobApplicationTests.java` and select "Run Java".

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "@SpringBootTest"
   description: "Right-click ➡️ Run Java"
   ```

   The test should pass which means we have the same result as before, but with less code!

Now if you run the test again, it should fail with the "Job Instance already exists and is complete" error. This is expected since we use the same shared database and we already have a job instance that was completed when we run the test for the first time. Let's fix that by clearing any job metadata before each test.
