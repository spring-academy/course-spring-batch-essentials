It is not uncommon for input data to have multiple errors. So, let's fix the file again!

1. Edit the input file... again!

   In the Editor, open `input/billing-2023-03.csv` and look at line 408:

   ```editor:open-file
   file: ~/exercises/input/billing-2023-03.csv
   line: 408
   description: "Open billing-2023-03.csv at line 408"
   ```

   ```csv
   2023,03,507,404-555-1407,36-07,507,216
   ```

   It looks like we have the same data format error.

   Correct the error by updating `36-07` to `36.07`.

   Now that the data is fixed, run the job again with the same command.

1. Rerun the job... again!

   When you rerun the job, the job should succeed. Great!

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   2023-08-14T18:05:45.486Z  INFO 4730 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{'data.year':'{value=2023, type=class java.lang.String, identifying=true}','output.file':'{value=staging/billing-report-2023-03.csv, type=class java.lang.String, identifying=true}','data.month':'{value=3, type=class java.lang.String, identifying=true}','input.file':'{value=input/billing-2023-03.csv, type=class java.lang.String, identifying=true}'}] and the following status: [COMPLETED] in 191ms
   ```

Next, let's inspect the metadata of Spring Batch to understand what happened with all of our job runs.
