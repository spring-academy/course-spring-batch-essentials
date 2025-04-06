Often, manually fixing a failed job's input file is the most pragmatic solution.

1. Edit the input file.

   In the `Editor`, open `input/billing-2023-03.csv` and change value `92-94` at line 226 to `92.94`.

   ```editor:open-file
   file: ~/exercises/input/billing-2023-03.csv
   line: 226
   description: "Open billing-2023-03.csv at line 226"
   ```

   Well, that was easy!

1. Rerun the job.

   Now that we have fixed the data, let's re-run the job with the following command:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-03.csv output.file=staging/billing-report-2023-03.csv data.year=2023 data.month=3
   ```

   So what happened this time?

   ```shell
   ... status: [FAILED]
   ```

   The job has failed again! Is it the same error?

1. Analyze the new failure.

   Let's inspect the error's stack trace:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   org.springframework.batch.item.file.FlatFileParseException: Parsing error at line: 408 in resource=[file [/home/eduk8s/exercises/input/billing-2023-03.csv]], input=[2023,03,507,404-555-1407,36-07,507,216]
   ...
   Caused by: org.springframework.beans.TypeMismatchException: Failed to convert value of type 'java.lang.String' to required type 'float'; For input string: "36-07"
   ...
   Caused by: java.lang.NumberFormatException: For input string: "36-07"
   ```

   The error is not the same as before, but it is similar: It seems that line 408 contains another invalid value for the `data usage` field, which is `36-07`.

We have yet another error. Let's repeat the process and fix it.
