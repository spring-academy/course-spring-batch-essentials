We need to make some updates to our billing job to make use of the new `PricingService`. Let's make those changes now.

1. Update the `BillingDataProcessor`.

   In the `Editor`, open the `src/main/java/example/billingjob/BillingDataProcessor.java` file and update its content as follows:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingDataProcessor.java
   description: "Open BillingDataProcessor.java"
   ```

   ```java
   package example.billingjob;

   import org.springframework.batch.item.ItemProcessor;
   import org.springframework.beans.factory.annotation.Value;

   public class BillingDataProcessor implements ItemProcessor<BillingData, ReportingData> {

   	private final PricingService pricingService;

   	public BillingDataProcessor(PricingService pricingService) {
   		this.pricingService = pricingService;
   	}

   	@Value("${spring.cellular.spending.threshold:150}")
   	private float spendingThreshold;

   	@Override
   	public ReportingData process(BillingData item) {
   		double billingTotal =
   				item.dataUsage() * pricingService.getDataPricing() +
   				item.callDuration() * pricingService.getCallPricing() +
   				item.smsCount() * pricingService.getSmsPricing();
   		if (billingTotal < spendingThreshold) {
   			return null;
   		}
   		return new ReportingData(item, billingTotal);
   	}
   }
   ```

   In this change, we have made the item processor use the `PricingService`. The calculation and filtering logic are the same, we just externalized the way to get pricing data by using the `PricingService`, which might fail!

   Let's try this out, but before that, we need to update the bean definition of the item processor by providing the pricing service.

1. Update the `BillingDataProcessor` bean.

   Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and update the method `billingDataProcessor` as follows:

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   text: "billingDataProcessor"
   description: "Open billingDataProcessor()"
   ```

   ```java
   @Bean
   public BillingDataProcessor billingDataProcessor(PricingService pricingService) {
   	return new BillingDataProcessor(pricingService);
   }
   ```

   In this snippet, we inject the pricing service in the `BillingDataProcessor` bean.

That's all! Let's run the job.
