One of the features of Spring Boot when it comes to supporting Spring Batch is the automatic execution of any `Job` bean defined in the application context at application startup.
So, to launch the `Job`, all you need is start the Spring Boot application.

Spring Boot looks for our `Job` in the application context and runs it by using the `JobLauncher`, which is autoconfigured and ready for us to use.

1. Run the application.

   To start the Spring Boot application, right-click on the `src/main/java/example/billingjob/BillingJobApplication.java` file in the `Editor` tab and select `Run Java`.

   ```editor:select-matching-text
   file: ~/exercises/src/main/java/example/billingjob/BillingJobApplication.java
   text: "@SpringBootApplication"
   description: "Right-click ➡️ Run Java"
   ```

   You should see something like the following output in the _TERMINAL_ tab of the `Editor`:

   ```shell
   .   ____          _            __ _ _
   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
   ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
   '  |____| .__|_| |_|_| |_\__, | / / / /
   =========|_|==============|___/=/_/_/_/
   :: Spring Boot ::                (v3.0.5)

   2023-04-28T08:28:50.992Z  INFO 7241 --- [           main] e.billingjob.BillingJobApplication       : Starting BillingJobApplication using Java 17.0.6 with PID 7241 (/home/eduk8s/exercises/target/classes started by eduk8s in /home/eduk8s/exercises)
   2023-04-28T08:28:50.995Z  INFO 7241 --- [           main] e.billingjob.BillingJobApplication       : No active profile set, falling back to 1 default profile: "default"
   2023-04-28T08:28:51.589Z  INFO 7241 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
   2023-04-28T08:28:51.831Z  INFO 7241 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@76f856a8
   2023-04-28T08:28:51.833Z  INFO 7241 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
   2023-04-28T08:28:51.995Z  INFO 7241 --- [           main] e.billingjob.BillingJobApplication       : Started BillingJobApplication in 1.362 seconds (process running for 1.699)
   2023-04-28T08:28:51.997Z  INFO 7241 --- [           main] o.s.b.a.b.JobLauncherApplicationRunner   : Running default command line with: []
   2023-04-28T08:28:52.049Z  INFO 7241 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [example.billingjob.BillingJob@73d4066e] launched with the following parameters: [{}]
   processing billing information
   2023-04-28T08:28:52.056Z  INFO 7241 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [example.billingjob.BillingJob@73d4066e] completed with the following parameters: [{}] and the following status: [STARTING]
   2023-04-28T08:28:52.068Z  INFO 7241 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
   2023-04-28T08:28:52.085Z  INFO 7241 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
   ```

   The `Job` has run correctly and completed successfully as we see the `processing billing information` message in the standard output as expected.

How can we verify that the `Job` ran successfully?

Let's do that next!
