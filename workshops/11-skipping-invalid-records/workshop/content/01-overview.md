In the previous Lab, you have seen that whenever an invalid line is encountered in the input file, the job fails immediately. Therefore, you had to fix the data and restart the job again and again until completion.

In this Lab, you will learn how to configure Spring Batch to skip invalid items and write them to a separate file for later analysis.

We will use the same problematic file, `input/billing-data-2023-03.csv`, as input to the job. Remember, that file contains two invalid lines: 226 and 408.

However, let's decide that the job should not fail any more and that the invalid lines should be written to a new file named `staging/billing-data-skip-2023-03.psv` so that we can audit the problematic records. This file is a Pipe Separated Values file and should contain invalid lines in the following format:

```csv
...
226|2023,03,325,404-555-1225,92-94,375,544
408|2023,03,507,404-555-1407,36-07,507,216
...
```

This simple format **starts with the skipped line number** followed by the raw value of the line from the original file.

Let's get started!
