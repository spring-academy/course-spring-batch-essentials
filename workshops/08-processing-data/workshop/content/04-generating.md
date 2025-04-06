As mentioned previously, the billing report is a flat file having the same format as the input file but with only a subset of customers and an additional column for the total billing amount.

To write data in a flat file, Spring Batch provides the `FlatFileItemWriter`, which we will use in this Lab.

Open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and add the following bean definition:

```editor:open-file
file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
description: "Open BillingJobConfiguration.java"
```

```java
@Bean
public FlatFileItemWriter<ReportingData> billingDataFileWriter() {
		return new FlatFileItemWriterBuilder<ReportingData>()
			.resource(new FileSystemResource("staging/billing-report-2023-01.csv"))
			.name("billingDataFileWriter")
			.delimited()
			.names("billingData.dataYear", "billingData.dataMonth", "billingData.accountId", "billingData.phoneNumber", "billingData.dataUsage", "billingData.callDuration", "billingData.smsCount", "billingTotal")
			.build();
}
```

You should also add the following import statements:

```java
import org.springframework.batch.item.file.FlatFileItemWriter;
import org.springframework.batch.item.file.builder.FlatFileItemWriterBuilder;
```

The `FlatFileItemWriter` needs to be configured with the target file, which is `staging/billing-report-2023-01.csv`. We also need to specify the format of the file (which is delimited in our case) and the fields we want to export in it. This is done using the `.delimited()` and `.names()` methods respectively.

This item writer accepts items of type `ReportingData` that were transformed from `BillingData` in the item processor. The list of fields coming from the nested `ReportingData.billingData` are prefixed with `billingData`.

The last field, `billingTotal`, does not require that prefix since it is defined directly in the `ReportingData` type.

That's all for the item writer, we can now proceed to the final step definition.
