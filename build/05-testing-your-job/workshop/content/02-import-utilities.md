The `@SpringBatchTest` annotation registers the `JobLauncherTestUtils` and `JobRepositoryTestUtils` as Spring beans in the test context, so we can autowire them in the test class and use them as needed.

So let's go ahead and add the annotation and utilities to our test class.

1. Add the `@SpringBatchTest` annotation.


   In the "Editor" tab, open the `src/test/java/example/billingjob/BillingJobApplicationTests.java` file and add the `@SpringBatchTest` annotation on the test class:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "BillingJobApplicationTests"
   description: "Open BillingJobApplicationTests.java"
   ```

   ```java
   @SpringBatchTest
   @SpringBootTest
   @ExtendWith(OutputCaptureExtension.class)
   class BillingJobApplicationTests {
   ...
   }
   ```

   You also need to add the corresponding import statement:

   ```java
   import org.springframework.batch.test.context.SpringBatchTest;
   ```

1. Autowire the test utilities.

   **Note:** If you are not familiar with the concept of _Autowiring_, we recommend you read the [Dependencies](https://docs.spring.io/spring-framework/reference/6.0/core/beans/dependencies.html) section of the [Spring Framework Core Technologies reference documentation](https://docs.spring.io/spring-framework/reference/6.0/core.html). The subsection [Autowiring Collaborators](https://docs.spring.io/spring-framework/reference/6.0/core/beans/dependencies/factory-autowire.html) explains _Autowiring_ along with its limitations.

   Update the test class and add the test utilities:

   ```java
   @SpringBootTest
   @SpringBatchTest
   @ExtendWith(OutputCaptureExtension.class)
   class BillingJobApplicationTests {

      @Autowired
      private JobLauncherTestUtils jobLauncherTestUtils;

      @Autowired
      private JobRepositoryTestUtils jobRepositoryTestUtils;
   ...
   }
   ```

   You need to add the following import statements as well:

   ```java
   import org.springframework.batch.test.JobLauncherTestUtils;
   import org.springframework.batch.test.JobRepositoryTestUtils;
   ```

With those utilities in place, we can use them in our `BillingJob` test.
