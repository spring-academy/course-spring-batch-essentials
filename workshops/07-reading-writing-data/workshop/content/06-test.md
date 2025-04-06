Now that the job has a second step that inserts data into the `BILLING_DATA` table, we need to update the test to assert that that table contains the data from the input file.

1. Update the test.

   Open the `src/test/java/example/billingjob/BillingJobApplicationTests.java` file and add the following assertions at the end of the `testJobExecution` test:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "testJobExecution"
   description: "Open testJobExecution()"
   ```

   ```java
   @Test
   void testJobExecution() throws Exception {

      // ...

      Assertions.assertEquals(1000, JdbcTestUtils.countRowsInTable(jdbcTemplate, "BILLING_DATA"));
   }
   ```

   This assertion uses the `JdbcTestUtils` from Spring Framework to count the number of rows in a given table. In our case, the input file contains 1000 records, so we need to make sure the table contains 1000 records once the job is run. Note that the `countRowsInTable` requires a `JdbcTemplate` to query the table. The `JdbcTemplate` is autoconfigured by Spring Boot and can be autowired in the test class. So let's add it:

   ```java
   @SpringBatchTest
   @SpringBootTest
   class BillingJobApplicationTests {

      @Autowired
      private JdbcTemplate jdbcTemplate;

      // the rest of the test class here

   }
   ```

   You also need to add the following import statements:

   ```java
   import org.springframework.jdbc.core.JdbcTemplate;
   import org.springframework.test.jdbc.JdbcTestUtils;
   ```

1. Run the test.

   It is now time to run the test. What do you think the results will be?

   In the **Terminal**, run the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
   ```

   When we run the test, we see the results are interesting! We actually have _too many records_. Why is this?

   ```shell
   ...
   org.opentest4j.AssertionFailedError: expected: <1000> but was: <2000>
        at example.billingjob.BillingJobApplicationTests.testJobExecution(BillingJobApplicationTests.java:54)
   ...
   ```

   We have too many records in the database because we are using the same database as we used in the previous step. When we ran the job, we already inserted data!

   Let's automatically clean up the database before running tests.

1. Set up the tests for a clean run.

   Finally, as with cleaning up the metadata before each test using the `JobRepositoryTestUtils`, we need to clean-up our _business data_ as well before each test.

   In our case, we need to delete all rows from the `BILLING_DATA` table before each test so that the assertion on record count always passes.

   Add the following instruction in the `setUp` method:

   ```editor:select-matching-text
   file: ~/exercises/src/test/java/example/billingjob/BillingJobApplicationTests.java
   text: "public void setUp"
   description: "Open setUp()"
   ```

   ```java
   @BeforeEach
   public void setUp() {
   	this.jobRepositoryTestUtils.removeJobExecutions();
   	JdbcTestUtils.deleteFromTables(this.jdbcTemplate, "BILLING_DATA");
   }
   ```

   Rerun the tests now and you'll see that they pass!

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
   ...
   [INFO] Results:
   [INFO]
   [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
   [INFO]
   [INFO] ------------------------------------------------------------------------
   [INFO] BUILD SUCCESS
   [INFO] ------------------------------------------------------------------------
   ```
