In the previous lesson, we discussed that a batch `Job` is an entity that encapsulates an entire batch process.
In this Lab, you will learn how to implement, run and test a batch `Job` in an application based on Spring Batch and Spring Boot.

We will create a `Job` implementation that prints `processing billing information` to the console.
You almost never have to implement the `Job` interface yourself, as Spring Batch provides a few implementations that are ready to use.

In a future lesson, we create all the steps that define our `Job` by using one of these implementations. For now, in order
to understand the responsibilities of a Spring Batch `Job`, we implement the `Job` interface as follows:

1. Create the `BillingJob` Class.

   In the `Editor` tab, go to the `src/main/java/example/billingjob` folder, create a new file named `BillingJob.java` and update its content with the following code:

   ```editor:append-lines-to-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJob.java
   description: "Create BillingJob.java"
   ```

   ```java
   package example.billingjob;

   import org.springframework.batch.core.Job;
   import org.springframework.batch.core.JobExecution;

   public class BillingJob implements Job {

       @Override
       public String getName() {
           return null;
       }

       @Override
       public void execute(JobExecution execution) {
           // TODO implement business logic
       }
   }
   ```

   We just created an empty implementation of the `Job` interface that we need to update with the business logic of the `Job`.

2. Name the `Job`.

   Give the `Job` a name by updating the `getName()` method as follows:

   ```java
   @Override
   public String getName() {
       return "BillingJob";
   }
   ```

3. Implement `execute`.

   Now let's implement the `execute` method by printing a message to the standard output:

   ```java
   @Override
   public void execute(JobExecution execution) {
       System.out.println("processing billing information");
   }
   ```

Great! We have finished our implementation of the first `Job`.
