Open the `src/test/java/example/billingjob/BillingJobApplicationTests.java` test class and update the `testJobExecution` test as follows:

```editor:select-matching-text
file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
text: "testJobExecution"
description: "Open testJobExecution()"
```

```java
@Test
void testJobExecution() throws Exception {
	// given
	JobParameters jobParameters = new JobParametersBuilder()
			.addString("input.file", "src/main/resources/billing-2023-01.csv")
			.toJobParameters();

	// when
	JobExecution jobExecution = this.jobLauncherTestUtils.launchJob(jobParameters);

	// then
	Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
	Assertions.assertTrue(Files.exists(Paths.get("staging", "billing-2023-01.csv")));
}
```

You should also add the following import statements:

```java
import java.nio.file.Files;
import java.nio.file.Paths;
```

In this test, we pass the input file as a job parameter and we expect the file to be present in the `staging` directory after running the job. As you see, we do not assert on the presence of `processing billing information from file /some/input/file` message in the standard output anymore, so you can safely remove the `@ExtendWith(OutputCaptureExtension.class)` declaration and the corresponding import statements.

### Run the tests

Now let's run this test.

In the **Terminal**, run the following command:

```dashboard:open-dashboard
name: Terminal
```

```shell
[~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
```

The test should pass, which means the step is doing what it is supposed to do.

The property `-Dspring.batch.job.enabled=false` disables the automatic execution of the job by Spring Boot. Without this property,
the job will be executed twice: a first time at the application's startup, and a second time during the test. This is obviously
not desired when running tests, hence the use of that property.
