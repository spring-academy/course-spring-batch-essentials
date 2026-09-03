In the previous lesson, you learned about the retry feature in Spring Batch and how to use it in a chunk-oriented step. In this Lab, you will implement a retry policy in the report generation step.

Remember, this step uses an item processor to calculate the total spending for each customer based on pricing data.

In this Lab, pricing data will be provided by a separate service, `PricingService`, which is not reliable! Let's learn how to deal with such services.
