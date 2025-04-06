Calculating the monthly spending and filtering customers can be done in an item processor.

To calculate the monthly spending, we need the pricing for data usage, calls and texts. In this Lab, and to keep things simple, we assume that data pricing is as follows:

- Call pricing: **$0.50** per minute
- Text/SMS pricing: **$0.10** each
- Data pricing: **$0.01** per MB

This pricing configuration might vary and should be externalized in configuration properties. For this reason, we will implement them as Spring configuration properties with the previous pricing details as default values. Finally, we will create a new type that represents reporting data, which includes all details from `BillingData` with a new field for the total billing amount.

1. Create the `ReportingData` type.

   In the `Editor`, create a new file named `ReportingData.java` in the `src/main/java/example/billingjob` folder with the following content:

   ```editor:append-lines-to-file
   file: ~/exercises/src/main/java/example/billingjob/ReportingData.java
   description: "Create ReportingData.java"
   ```

   ```java
   package example.billingjob;

   public record ReportingData(BillingData billingData, double billingTotal) {
   }
   ```

   This is a Java record that contains billing information of customers as well as a new field named `billingTotal` which represents the monthly spending for each customer. We will calculate and fill in this field in an item processor.

1. Create the `BillingDataProcessor`.

   In the `Editor`, open the `src/main/java/example/billingjob` folder and create a new file named `BillingDataProcessor.java` with the following content:

   ```editor:append-lines-to-file
   file: ~/exercises/src/main/java/example/billingjob/BillingDataProcessor.java
   description: "Create BillingDataProcessor.java"
   ```

   ```java
   package example.billingjob;

   import org.springframework.batch.item.ItemProcessor;
   import org.springframework.beans.factory.annotation.Value;

   public class BillingDataProcessor implements ItemProcessor<BillingData, ReportingData> {

   	@Value("${spring.cellular.pricing.data:0.01}")
   	private float dataPricing;

   	@Value("${spring.cellular.pricing.call:0.5}")
   	private float callPricing;

   	@Value("${spring.cellular.pricing.sms:0.1}")
   	private float smsPricing;

   	@Value("${spring.cellular.spending.threshold:150}")
   	private float spendingThreshold;

      @Override
      public ReportingData process(BillingData item) {
         double billingTotal = item.dataUsage() * dataPricing + item.callDuration() * callPricing + item.smsCount() * smsPricing;
         if (billingTotal < spendingThreshold) {
            return null;
         }
         return new ReportingData(item, billingTotal);
      }
   }
   ```

   That's more code than we usually add! Let's dive into it.

1. Understand how the item processor works.

   Our new processor class implements the `ItemProcessor<BillingData,ReportingData>` interface.

   In this class, we calculate the monthly spending for the current customer and filter out customers who have spent less than $150 (by returning `null` in that case).

   Pricing details are declared as Spring properties under the `spring.cellular.*` namespace, and which can be configured externally. Those properties have default values which we are going to use in this Lab.

   Once the calculation and filtering are done, we create a `ReportingData` object with the billing information as well as the monthly spending.

   This item processor is a combination of different types of processing: it not only _enriches_ the current item with new data, but also _filters out_ items which are not needed for the current business requirement.

1. Declare the `BillingDataProcessor` as a bean.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   @Bean
   public BillingDataProcessor billingDataProcessor() {
   	return new BillingDataProcessor();
   }
   ```

   It is necessary to declare the `BillingDataProcessor` as a bean so that it is managed by Spring and configured with the properties we declared in it.

With our new processor implementation and bean definition in place, only customers who spent more than $150 will be passed to the item writer and included in the final report. So let's go ahead and define the writer.
