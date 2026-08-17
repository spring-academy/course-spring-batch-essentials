We are now finally ready to run the entire job and check the generated billing report!

1. Package and run.

   In the **Terminal** tab, run the following command to build the project:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Then, launch the job and pass the input file as a parameter with the following command:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2023-01.csv
   ```

   You should see something like the following log in the console:

   ```shell
   2023-07-26T11:34:59.072+02:00  INFO 97689 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=BillingJob]] launched with the following parameters: [{'input.file':'{value=src/main/resources/billing-2023-01.csv, type=class java.lang.String, identifying=true}'}]
   2023-07-26T11:34:59.128+02:00  INFO 97689 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [filePreparation]
   2023-07-26T11:34:59.172+02:00  INFO 97689 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [filePreparation] executed in 43ms
   2023-07-26T11:34:59.209+02:00  INFO 97689 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [fileIngestion]
   2023-07-26T11:34:59.490+02:00  INFO 97689 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [fileIngestion] executed in 280ms
   2023-07-26T11:34:59.519+02:00  INFO 97689 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [reportGeneration]
   2023-07-26T11:34:59.811+02:00  INFO 97689 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [reportGeneration] executed in 291ms
   2023-07-26T11:34:59.862+02:00  INFO 97689 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{'input.file':'{value=src/main/resources/billing-2023-01.csv, type=class java.lang.String, identifying=true}'}] and the following status: [COMPLETED] in 768ms
   ```

   Note how all three steps have been completed successfully. This means we should find the generated report in the `staging` directory. Let's check it out!

1. Check out the report.

   In the Editor, open the `staging` directory and inspect its content. A new file named `billing-report-2023-01.csv` should be generated in that folder.

   This file contains only customers who spent more than $150 per month. This file also contains an _additional column at the end of each line_ with the monthly spending for each customer. All values here should be greater than $150.

   ```csv
   2023,1,101,404-555-1001,69.87,289,77,152.89869689941406
   2023,1,104,404-555-1004,18.39,926,438,506.98388671875
   2023,1,106,404-555-1006,94.82,526,198,283.7481994628906
   2023,1,108,404-555-1008,74.8,533,926,359.8479919433594
   2023,1,109,404-555-1009,24.58,508,850,339.24578857421875
   ...
   ```

The number of customers in this file should be exactly 781. Let's update the test with that information.
