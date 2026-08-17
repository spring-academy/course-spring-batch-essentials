In the previous lesson, you learned about the chunk-oriented processing model and how Spring Batch reads and writes data. In this Lab, you will implement a chunk-oriented step that reads billing data from the input file and writes it into a relational database table. Remember, the second step of our `billingJob` is designed to save billing data in the database for later use.

Before moving to the step definition, let's start by understanding the specification of the billing data that should be parsed from the input file.

## Understanding the billing data specification

The input file in `src/main/resources/billing-2023-01.csv` has the following format:

| Column             | Type   | Description                                                   |
| ------------------ | ------ | ------------------------------------------------------------- |
| Year               | int    | The year during which the data was captured for the customer  |
| Month              | int    | The month during which the data was captured for the customer |
| Account Identifier | int    | The account identifier associated with the customer           |
| Phone Number       | String | The phone number that the usage is associated with            |
| Data Usage         | float  | The sum of data used for the month in MB                      |
| Call Duration      | int    | The sum of calls used for the month in minutes                |
| SMS Count          | int    | The number of text messages sent during the month             |

Here is a sample from the input file, which you can also check by opening the `src/main/resources/billing-2023-01.csv` file in the **Editor**:

```editor:open-file
file: ~/exercises/src/main/resources/billing-2023-01.csv
description: "Open billing-2023-01.csv"
```

| Year | Month | Account Number | Phone Number | Data Usage | Call Duration | SMS Count |
| ---- | ----- | -------------- | ------------ | ---------- | ------------- | --------- |
| 2023 | 1     | 100            | 404-555-1234 | 12.5       | 0             | 3         |
| 2023 | 1     | 101            | 404-555-5678 | 7.8        | 5             | 1         |
| 2023 | 1     | 102            | 404-555-9101 | 45.2       | 2             | 0         |
| 2023 | 1     | 104            | 404-555-2345 | 22.1       | 0             | 1         |
| 2023 | 1     | 105            | 404-555-6789 | 8.9        | 10            | 0         |
| 2023 | 1     | 106            | 404-555-1112 | 3.2        | 3             | 2         |
| 2023 | 1     | 107            | 404-555-1314 | 17.6       | 0             | 4         |
| 2023 | 1     | 108            | 404-555-1516 | 28.9       | 7             | 1         |
| 2023 | 1     | 109            | 404-555-1718 | 11.3       | 2             | 3         |

Now that we understand the format of the flat file we need to parse, we can define a `FlatFileItemReader` to read data from that file.
