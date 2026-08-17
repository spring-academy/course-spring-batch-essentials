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

You should see the `billing-2023-01.csv` file copied in the `staging` directory, which means that our job is running as expected.

Let's now check the metadata about the first step that Spring Batch recorded in the `BATCH_STEP_EXECUTION` table.

In the **Terminal**, run the following command:

```shell
[~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_STEP_EXECUTION;'
```

Examine the output to check the status of the `filePreparation` step, its start time and end time, etc.
