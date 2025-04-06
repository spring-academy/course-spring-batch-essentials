In the previous lesson, you learned about the different use cases of data processing with Spring Batch and how to use an `ItemProcessor` in a chunk-oriented step. In this Lab, you will implement the last step of the `BillingJob`, which consists of generating the billing report.

To keep things simple, the billing report is actually a subset of the input file with customers who spent more than $150 per month. The format of the output file is the same as the input file, with an additional column representing the total billing amount for the month.

For this last step, you need to:

- Read data from the `BILLING_DATA` table, which was filled in earlier by the `fileIngestion` step
- Calculate the monthly spending of each customer
- Filter out customers who spent less than $150
- And finally generate a flat file with the remaining customers

Let's get started!
