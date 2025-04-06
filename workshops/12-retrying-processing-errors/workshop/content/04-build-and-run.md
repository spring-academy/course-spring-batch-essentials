As we've done many times before in previous labs, let's build, run, and investigate the results.

1. Build and run the job.

   First, open a **Terminal** and build the new version of the job:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Now, let's run the job on the `input/billing-2023-04.csv` file.

   This is a **_new file_** that represents the billing data for April 2023. It contains 200 lines of billing information. All lines are correctly formatted this time.

   In the **Terminal**, run the following command:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-04.csv output.file=staging/billing-report-2023-04.csv skip.file=staging/billing-data-skip-2023-04.psv data.year=2023 data.month=4
   ```

   The job failed! Let's check why.

1. Investigate the failure.

   The error should be similar to the following:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   example.billingjob.PricingException: Error while retrieving data pricing
     at example.billingjob.PricingService.getDataPricing(PricingService.java:22)
   at example.billingjob.BillingDataProcessor.process(BillingDataProcessor.java:20)
   ```

   The item processor failed to calculate the total spending due to an exception thrown by the pricing service.

   What should we do now? If you try to run the job again, it fails again!

As you can see, there is no guarantee that the job succeeds in each attempt, and trying again and again is definitely not the way to go!

It looks like we need a retry strategy.
