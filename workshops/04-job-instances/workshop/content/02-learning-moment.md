As we saw from the simple message we print to the console, the first `JobInstance` corresponding to the processing of the `billing-2023-01.csv` file has completed successfully and the billing report was generated correctly.

Running the same `JobInstance` again would be a waste of resources. But what if we **_accidentally_** re-run the same file again?

How might this happen? Sometimes re-running the same job is due to human error. Other times a technical issue or a platform limitation might trigger a re-run. Whatever the reason, a `JobInstance` should not result in dire concequences if it is run more than once. Nobody wants to be double-billed for their cellular phone usage!

So let's try that and see what happens!

In the initial **Terminal** tab, run the following command:

```dashboard:open-dashboard
name: Terminal
```

```shell
[~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=src/main/resources/billing-2023-01.csv
```

You should see an error as follows:

```shell
org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException: A job instance already exists and is complete
   for parameters={'input.file':'{value=src/main/resources/billing-2023-01.csv, type=class java.lang.String, identifying=true}'}.
   If you want to run this job again, change the parameters.
```

As you can see, with no further configuration, Spring Batch prevented the same `JobInstance` from running a second time. This default design choice addresses the human errors and platform limitations we mentioned earlier.

After that failure in re-launching a successful `JobInstance`, the `BATCH_JOB_EXECUTION` should **_not_** contain another execution, which you can check with the following command:

```shell
[~/exercises] $ docker exec postgres psql -U postgres -c 'select count(*) from BATCH_JOB_EXECUTION;'
count
-------
     1
(1 row)
```

Now that we are protected against re-running a `JobInstance`, let's see what happens when we run a second, different `JobInstance`.
