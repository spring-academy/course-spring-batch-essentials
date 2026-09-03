In the previous lesson, you learned how to read and write data and how to configure a chunk-oriented step. You implemented the second step of the `BillingJob` in the previous Lab, and ingested data from the input file into the relational database table without any modification.

While it's common to move data around, it's rarely done without modifying it, adapting it, or processing it in some way. In fact, input/output systems aren't always a perfect match. As a batch developer, you'll have to find a way to process data within a chunk-oriented step.

In this lesson, you'll learn how to process data in Spring Batch. You'll learn the main API for data processing, and how to use it within a chunk-oriented step.

## The `ItemProcessor` API

Processing items in a chunk-oriented step happens between reading and writing data. It's an optional phase of the chunk-oriented processing model, where items returned by the reader are processed, then handed over to the writer. You can think of it as an intermediate stage in a processing pipeline.

Processing items in Spring Batch is done by implementing the `ItemProcessor` interface, which is defined as follows:

```java
@FunctionalInterface
public interface ItemProcessor<I, O> {

   @Nullable
   O process(@NonNull I item) throws Exception;
}
```

This is a functional interface with a single method `process`. This method takes an item of type `I` as input and returns an item of type `O` as output. The method name `process` is generic on purpose as processing data is a broad term and covers different use cases like transforming data, enriching it, validating it, or filtering the data.

Each outcome from this method corresponds to a particular use case which we'll cover in the next sections.

## Transforming Data

The `process` method takes an item of type `I` as input and returns an item of type `O` as output. This clear distinction between input and output types is designed to allow developers to change the item type during the processing phase. This is useful when adapting the input data to the format expected by the target system.

Here's an example of an item processor that transforms items from a hypothetical type `BillingData` to another type `ReportingData`:

```java
public class BillingDataProcessor implements ItemProcessor<BillingData, ReportingData> {

   public ReportingData process(BillingData item) {
       return new ReportingData(item);
   }
}
```

While it's possible to change the item type during processing, what if we don't need that? Meaning, what if the input type `I` is the same as the output type `O`? This is what we'll discuss in the next section.

## Enriching Data

Transforming data is not the only use case of an `ItemProcessor`. In fact, items aren't necessarily transformed from one type to another.

In many batch processing jobs, the requirement is to enrich input data with additional details before saving it to the target system. In this case, items aren't transformed from one type to another, but enriched with additional information. For instance, an item processor can request an external system to enrich the current item, then return the enriched item:

```java
public class EnrichingItemProcessor implements ItemProcessor<Person, Person> {

   private AddressService addressService;

   public EnrichingItemProcessor(AddressService addressService) {
      this.addressService = addressService;
   }

   public Person process(Person person) {
      Address address = this.addressService.getAddress(person);
      person.setAddress(address);
      return person;
   }
}
```

In this `EnrichingItemProcessor`, items of type `Person` are enriched with the person's address using a hypothetical `AddressService`. This is a typical example of an item processor that enriches data without transforming it.

## Validating Data

The `process` method is designed to throw an exception in case of a processing error.
Processing errors could be technical errors (like a failure to call an external service) or functional errors (like invalid items).

One of the most common use cases of an item processor is data validation. Once we read items from an input source, we might need to validate if the input data is valid or not before saving it to the target system. In typical enterprise systems, data is often consumed from a trusted source, but this isn't always the case. In fact, in many enterprise batch systems, data is consumed from external sources which might not be trustworthy, in which case, validating data is crucial to the security and integrity of the target system.

An `ItemProcessor` is the ideal place to implement data validation rules. Here's an example:

```java
public class ValidatingItemProcessor implements ItemProcessor<Person, Person> {

   private EmailService emailService;

   public ValidatingItemProcessor(EmailService emailService) {
      this.emailService = emailService;
   }

   public Person process(Person person) {
      if (!this.emailService.isValid(person.getEmail()) {
         throw new InvalidEmailException("Invalid email for " + person);
      }
      return person;
   }
}
```

This `ValidatingItemProcessor` is designed to validate the person's email using a hypothetical `EmailService`, and reject invalid items by throwing an `InvalidEmailException`. On the other hand, if the person's email is valid, the item is returned as-is.

## Filtering Data

The last outcome from the `process` method that we haven't addressed yet, is when the method returns `null`. How does Spring Batch interpret such a result from the item processor?

Returning `null` from the `process` method tells Spring Batch to _filter out the current item_. Filtering the current item simply means to not let it continue in the processing pipeline. Therefore, it'll be excluded from being written as part of the current chunk. You can think of this like Unix filters or Java's `Stream#filter` operation.

Data filtering is another typical use case for an item processor and which we'll use in the Lab of this lesson. In fact, the last step of our `BillingJob` is to generate a billing report for customers who spent more than $150 per month. This means we should filter out all customers who spent less than that amount. We'll implement this business rule in the Lab of this lesson, but here's a typical item processor for that particular case:

```java
public class FilteringItemProcessor implements ItemProcessor<BillingData, BillingData> {

   public BillingData process(BillingData item) }
      if (item.getMonthlySpending() < 150) {
         return null; // filter customers spending less than $150
      }
      return item;
   }
}
```

In this example, items with a `monthlySpending` amount less than $150 are filtered and won't be included in the final report.
