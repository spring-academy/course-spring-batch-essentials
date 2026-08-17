In addition to the previous verifications, we need to add two new assertions:

- We need to assert that the billing report file exists in the `staging` directory
- And that the report contains exactly 781 lines

Let's update the test accordingly.

1. Update the tests.
   Open the `src/test/java/example/billingjob/BillingJobApplicationTests.java` file and add the following lines at the end of the `testJobExecution` test method:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "testJobExecution"
   description: "Open testJobExecution()"
   ```

   ```java
   @Test
   void testJobExecution() throws Exception {
      // ...
      Path billingReport = Paths.get("staging", "billing-report-2023-01.csv");
      Assertions.assertTrue(Files.exists(billingReport));
      Assertions.assertEquals(781, Files.lines(billingReport).count());
   }
   ```

   You should also add the following import statement:

   ```java
   import java.nio.file.Path;
   ```

   These assertions implement the verifications we described above. Now it is time to run the test and check the result. Open a **Terminal** and run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
   ```

   The test should pass, which means the third step and the entire job is doing what it is supposed to do. Congratulations!
