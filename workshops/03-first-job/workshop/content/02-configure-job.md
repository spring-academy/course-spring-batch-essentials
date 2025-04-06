Now that we implemented our first `Job`, let's declare it as a bean in the application context so that Spring Boot detects it and runs it on our application startup.

Spring uses a class for Java-based configuration by adding the `@Configuration` annotation. This class is used as a source
of bean definitions. In this section, we create a Spring configuration class that contains our `Job` bean definition.

1. Configure the `Job` bean.

   In the `Editor` tab, open the `src/main/java/example/billingjob/BillingJobConfiguration.java` file and update its content with the following code:

   ```editor:open-file
   file: ~/exercises/src/main/java/example/billingjob/BillingJobConfiguration.java
   description: "Open BillingJobConfiguration.java"
   ```

   ```java
   package example.billingjob;

   import org.springframework.batch.core.Job;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;

   @Configuration
   public class BillingJobConfiguration {
       @Bean
       public Job job() {
           return new BillingJob();
       }
   }
   ```

We are now ready to run our `Job`, so let's run it and see what happens!
