As with the success path, it is the `Job`'s responsibility to handle runtime exceptions and report its failure to the `JobRepository`.

As a further exercise, try to simulate an exception in the `execute` method and report a `FAILED` status to the repository.

Note how the `execute` method is _not_ expected to throw exceptions, and it is the `Job` implementation's responsibility to handle them and add them to the `JobExecution` object for later inspection:

```editor:select-matching-text
file: ~/exercises/src/main/java/example/billingjob/BillingJob.java
text: "public void execute"
description: "Open BillingJob.java"
```

```java
@Override
public void execute(JobExecution execution) {
    try {
        throw new Exception("Unable to process billing information");
    } catch (Exception exception) {
        execution.addFailureException(exception);
        execution.setStatus(BatchStatus.COMPLETED);
        execution.setExitStatus(ExitStatus.FAILED.addExitDescription(exception.getMessage()));
    } finally {
        this.jobRepository.update(execution);
    }
}
```

Give this a try:

1. Clean up the database using the `scripts/drop-create-database.sh` script as shown above.
2. Re-run the `Job` with the intentionally-failing business logic.

In addition to the `status` of `COMPLETED` status, you should also an `exit_code` of `FAILED` status in the database as well as the error message in the `exist_message` column.

```dashboard:open-dashboard
name: Terminal
```

```shell
[~/exercises] $ docker exec postgres psql -U postgres -c 'select * from BATCH_JOB_EXECUTION;'
 job_execution_id | version | job_instance_id |        create_time         | start_time | end_time |  status   | exit_code |             exit_message              |        last_updated
------------------+---------+-----------------+----------------------------+------------+----------+-----------+-----------+---------------------------------------+----------------------------
                1 |       1 |               1 | 2023-05-03 21:30:29.142053 |            |          | COMPLETED | FAILED    | Unable to process billing information | 2023-05-03 21:30:29.151454
(1 row)
```

**Note:** be sure to revert the `execute` method back to the successful implementation before continuing.
