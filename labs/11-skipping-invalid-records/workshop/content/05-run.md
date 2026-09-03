Let's see if our fault-tolerant job is truly fault-tolerant!

1. Package and run.

   Open the **Terminal** and run the following command to build the project:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Then, run the job to process `billing-report-2026-03.csv`:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2026-03.csv output.file=staging/billing-report-2026-03.csv skip.file=staging/billing-data-skip-2026-03.csv data.year=2026 data.month=3
   ```

   **Note** the new parameter `skip.file=staging/billing-data-skip-2026-03.csv` that corresponds to the skipped items file.

   The job should succeed and a new file named `billing-data-skip-2026-03.csv` should be generated in the `staging` directory.

   ```shell
   ...
   2026-08-27T12:31:58.562Z  INFO 2797 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{JobParameter{name='data.year', value=2026, type=class java.lang.String, identifying=true},JobParameter{name='output.file', value=staging/billing-report-2026-03.csv, type=class java.lang.String, identifying=true},JobParameter{name='data.month', value=3, type=class java.lang.String, identifying=true},JobParameter{name='skip.file', value=staging/billing-data-skip-2026-03.csv, type=class java.lang.String, identifying=true},JobParameter{name='input.file', value=input/billing-2026-03.csv, type=class java.lang.String, identifying=true}}] and the following status: [COMPLETED] in 801ms
   ```

   Let's check it out!

1. Inspect the skipped items.

   In the `Editor`, open the `staging/billing-data-skip-2026-03.csv` and check its content. It should contain the following lines:

   ```editor:open-file
   file: ~/exercises/staging/billing-data-skip-2026-03.csv
   description: "Open billing-2026-03.csv"
   ```

   ```csv
   226|2026,03,325,404-555-1225,92-94,375,544
   408|2026,03,507,404-555-1407,36-07,507,216
   ```

   This means our job has correctly skipped incorrect lines without failing every time! That's great compared to the previous way of fixing data line by line and restarting the job again and again until completion.

   With that new file containing skipped lines, we can decide how to proceed. We can either fix lines all at once and ingest that file separately or just ignore them if they contain bad billing data that cannot be used.
