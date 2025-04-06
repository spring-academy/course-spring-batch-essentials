Let's see if our fault-tolerant job is truly fault-tolerant!

1. Package and run.

   Open the **Terminal** and run the following command to build the project:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Then, run the job to process `billing-report-2023-03.csv`:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-03.csv output.file=staging/billing-report-2023-03.csv skip.file=staging/billing-data-skip-2023-03.psv data.year=2023 data.month=3
   ```

   **Note** the new parameter `skip.file=staging/billing-data-skip-2023-03.psv` that corresponds to the skipped items file.

   The job should succeed and a new file named `billing-data-skip-2023-03.psv` should be generated in the `staging` directory.

   ```shell
   ...
   XXXX-08-16T00:43:39.917Z  INFO 2834 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{'data.year':'{value=2023, type=class java.lang.String, identifying=true}','output.file':'{value=staging/billing-report-2023-03.csv, type=class java.lang.String, identifying=true}','data.month':'{value=3, type=class java.lang.String, identifying=true}','skip.file':'{value=staging/billing-data-skip-2023-03.psv, type=class java.lang.String, identifying=true}','input.file':'{value=input/billing-2023-03.csv, type=class java.lang.String, identifying=true}'}] and the following status: [COMPLETED] in 285ms
   ```

   Let's check it out!

1. Inspect the skipped items.

   In the `Editor`, open the `staging/billing-data-skip-2023-03.psv` and check its content. It should contain the following lines:

   ```editor:open-file
   file: ~/exercises/staging/billing-data-skip-2023-03.psv
   description: "Open billing-2023-03.csv"
   ```

   ```csv
   226|2023,03,325,404-555-1225,92-94,375,544
   408|2023,03,507,404-555-1407,36-07,507,216
   ```

   This means our job has correctly skipped incorrect lines without failing every time! That's great compared to the previous way of fixing data line by line and restarting the job again and again until completion.

   With that new file containing skipped lines, we can decide how to proceed. We can either fix lines all at once and ingest that file separately or just ignore them if they contain bad billing data that cannot be used.
