As a further exercise, try the following scenarios:

### Omit the input file

In the `FilePreparationTasklet`, what if the job parameter `input.file` is not passed to the job as expected? You can update the implementation to check the existence of that mandatory parameter and throw an exception to make the tasklet (and the enclosing step and job) fail.

You can also set a `JobParametersValidator` on the job to validate the existence of the `input.file` parameter. Feel free to review `JobParametersValidator` [in the documentation](https://docs.spring.io/spring-batch/docs/5.0.4/reference/html/job.html#jobparametersvalidator).

### Force an exception

In the `FilePreparationTasklet`, the `Files.copy` operation will throw an exception if the input file is not present.

Try to rename or remove the `src/main/resources/billing-2023-01.csv`, then run the job and see the result.
