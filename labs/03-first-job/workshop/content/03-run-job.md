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

    :: Spring Boot ::                (v4.1.0)

   2026-08-18T10:49:15.668Z  INFO 1841 --- [           main] e.billingjob.BillingJobApplication       : Starting BillingJobApplication using Java 17.0.16 with PID 1841 (/home/eduk8s/exercises/target/classes started by eduk8s in /home/eduk8s/exercises)
   2026-08-18T10:49:15.676Z  INFO 1841 --- [           main] e.billingjob.BillingJobApplication       : No active profile set, falling back to 1 default profile: "default"
   2026-08-18T10:49:17.705Z  INFO 1841 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
   2026-08-18T10:49:18.268Z  INFO 1841 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@2f651f93
   2026-08-18T10:49:18.273Z  INFO 1841 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
   2026-08-18T10:49:18.544Z  INFO 1841 --- [           main] e.billingjob.BillingJobApplication       : Started BillingJobApplication in 3.782 seconds (process running for 4.398)
   2026-08-18T10:49:18.550Z  INFO 1841 --- [           main] o.s.b.b.a.JobLauncherApplicationRunner   : Running default command line with: []
   2026-08-18T10:49:18.788Z  INFO 1841 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [example.billingjob.BillingJob@3b705be7] launched with the following parameters: [{}]
   processing billing information
   2026-08-18T10:49:18.795Z  INFO 1841 --- [           main] o.s.b.c.l.s.TaskExecutorJobLauncher      : Job: [example.billingjob.BillingJob@3b705be7] completed with the following parameters: [{}] and the following status: [STARTING]
   2026-08-18T10:49:18.830Z  INFO 1841 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
   2026-08-18T10:49:18.862Z  INFO 1841 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
   ```

   The `Job` has run correctly and completed successfully as we see the `processing billing information` message in the standard output as expected.

How can we verify that the `Job` ran successfully?

Let's do that next!
