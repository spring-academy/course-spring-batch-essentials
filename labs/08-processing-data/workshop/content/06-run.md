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
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2026-01.csv
   ```

   You should see something like the following log in the console:

   ```shell
   2026-08-27T11:01:09.814Z  INFO 5221 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=BillingJob]] launched with the following parameters: [{JobParameter{name='input.file', value=src/main/resources/billing-2026-01.csv, type=class java.lang.String, identifying=true}}]
2026-08-27T11:01:09.882Z  INFO 5221 --- [           main] o.s.batch.core.step.AbstractStep         : Executing step: [filePreparation]
2026-08-27T11:01:09.908Z  INFO 5221 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [filePreparation] executed in 31ms
2026-08-27T11:01:09.949Z  INFO 5221 --- [           main] o.s.batch.core.step.AbstractStep         : Executing step: [fileIngestion]
2026-08-27T11:01:10.448Z  INFO 5221 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [fileIngestion] executed in 505ms
2026-08-27T11:01:10.486Z  INFO 5221 --- [           main] o.s.batch.core.step.AbstractStep         : Executing step: [reportGeneration]
2026-08-27T11:01:10.838Z  INFO 5221 --- [           main] o.s.batch.core.step.AbstractStep         : Step: [reportGeneration] executed in 355ms
2026-08-27T11:01:10.875Z  INFO 5221 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [SimpleJob: [name=BillingJob]] completed with the following parameters: [{JobParameter{name='input.file', value=src/main/resources/billing-2026-01.csv, type=class java.lang.String, identifying=true}}] and the following status: [COMPLETED] in 1s36ms
   ```

   Note how all three steps have been completed successfully. This means we should find the generated report in the `staging` directory. Let's check it out!

1. Check out the report.

   In the Editor, open the `staging` directory and inspect its content. A new file named `billing-report-2026-01.csv` should be generated in that folder.

   This file contains only customers who spent more than $150 per month. This file also contains an _additional column at the end of each line_ with the monthly spending for each customer. All values here should be greater than $150.

   ```csv
   2026,1,101,404-555-1001,69.87,289,77,152.89869689941406
   2026,1,104,404-555-1004,18.39,926,438,506.98388671875
   2026,1,106,404-555-1006,94.82,526,198,283.7481994628906
   2026,1,108,404-555-1008,74.8,533,926,359.8479919433594
   2026,1,109,404-555-1009,24.58,508,850,339.24578857421875
   ...
   ```

The number of customers in this file should be exactly 781. Let's update the test with that information.
