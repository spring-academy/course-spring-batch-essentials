For this lab, we have prepared a class in `src/main/java/example/billingjob/PricingService.java` that provides pricing information. Pricing data is available through getters for data, calls and sms.

Please open that class now and take a look at it.

Let's focus on one method in particular: `getDataPricing()`:

```editor:select-matching-text
file: ~/exercises/src/main/java/example/billingjob/PricingService.java
text: "getDataPricing"
description: "Open getDataPricing()"
```

```java
public class PricingService {
  ...
	public float getDataPricing() {
		if (this.random.nextInt(1000) % 7 == 0) {
			throw new PricingException("Error while retrieving data pricing");
		}
		return this.dataPricing;
	}
}
```

To simulate failures, `getDataPricing()` will fail randomly by throwing a `PricingException`.

On a real project, this service could have been a flaky web service that might fail due to network errors, but to keep this lab simple, we made this service local. In both cases, the result would have been the same.

For your convenience, we also declared this service as a bean in the `src/main/java/example/billingjob/BillingJobConfiguration` class:

```editor:select-matching-text
file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
text: "pricingService"
description: "Open pricingService()"
```

```java
@Bean
public PricingService pricingService() {
	return new PricingService();
}
```

This is required to be able to configure pricing data though Spring properties, as we did previously for the `BillingDataProcessor`.
