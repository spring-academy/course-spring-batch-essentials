Now that our Job can take input values at runtime, let's generate billing reports for both of our data input files, `billing-2023-01.csv` and `billing-2023-02.csv`.

1. Generate the `2023-01` report.

   In the **Terminal** tab, run the following command to build the project:

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ ./mvnw clean package -Dmaven.test.skip=true
   ```

   Now you can run the job with the following command for `billing-2023-01.csv`:

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-01.csv output.file=staging/billing-report-2023-01.csv data.year=2023 data.month=1
   ```

   Notice that you're now passing in 4 parameters:

   - `input.file`
   - `output.file`
   - `data.year`
   - `data.month`

   What would happen if you omitted any of these? Keep that in mind and we'll try it later!

   For now, you should see the same result as in the previous lab -- the input and output files for `2023-01` in the `staging` directory:

   ```shell
   staging/
   - billing-2023-01.csv
   - billing-report-2023-01.csv
   ```

   **_Note:_** we moved input files from the `src/main/resources` directory to the `input` directory. Resources under `src/main/resources` are included by Maven in the final jar by default, and we obviously do not want input files to be included in the jar of our job.

   But what should happen if we run the job and provide `billing-2023-02.csv` as the input data? Let's find out!

1. Generate the `2023-02` report.

   In the **Terminal** tab, run the job, but provide the input and output values for the `2023-02` data.

   What do you the output will be?

   ```dashboard:open-dashboard
   name: Terminal
   ```

   ```shell
   [~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-02.csv output.file=staging/billing-report-2023-02.csv data.year=2023 data.month=2
   ```

   Looking in `staging/` you can see we now have _both_ sets of input and output data: `2023-01` _and_ `2023-02`!

   ```shell
   staging/
   - billing-2023-01.csv
   - billing-2023-02.csv
   - billing-report-2023-01.csv
   - billing-report-2023-02.csv
   ```

### Learning moment: missing parameters

Similar to what we have mentioned in the "Implementing the File Preparation Step" Lab, if job parameters are not specified, then the configuration of step-scoped beans will fail, which in turn will fail the job.
Give it a try. In the **Terminal**, try omitting the `output.file` parameter and observe the `Path must not be null` error:

```dashboard:open-dashboard
name: Terminal
```

```shell
[~/exercises] $ java -jar target/billing-job-0.0.1-SNAPSHOT.jar input.file=input/billing-2023-02.csv
...
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'scopedTarget.billingDataFileWriter' defined in class path resource [example/billingjob/BillingJobConfiguration.class]: Failed to instantiate [org.springframework.batch.item.file.FlatFileItemWriter]: Factory method 'billingDataFileWriter' threw exception with message: Path must not be null
```

Always make sure to pass job parameters as intended or validate that by registering a `JobParametersValidator` in the job definition.
