Now that the structure of the application is in place, let's check if the application is correctly configured and running as expected.

In the editor tab, right-click on the `billing-job/src/main/java/example/billingjob/BillingJobApplication.java` file and select `Run Java`.

```editor:select-matching-text
file: ~/exercises/billing-job/src/main/java/example/billingjob/BillingJobApplication.java
text: "@SpringBootApplication"
description: "Right-click ➡️ Run Java"
```

You should see something like the following output in the Editor's _TERMINAL_ tab at the bottom of the screen:

```shell
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.0.5)

2023-04-13T12:06:41.998Z  INFO 16276 --- [           main] e.billingjob.BillingJobApplication       : Starting BillingJobApplication using Java 17.0.6 with PID 16276 (/home/eduk8s/exercises/billing-job/target/classes started by eduk8s in /home/eduk8s/exercises)
2023-04-13T12:06:42.002Z  INFO 16276 --- [           main] e.billingjob.BillingJobApplication       : No active profile set, falling back to 1 default profile: "default"
2023-04-13T12:06:42.606Z  INFO 16276 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-04-13T12:06:42.824Z  INFO 16276 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@45667d98
2023-04-13T12:06:42.825Z  INFO 16276 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2023-04-13T12:06:42.959Z  INFO 16276 --- [           main] e.billingjob.BillingJobApplication       : Started BillingJobApplication in 1.391 seconds (process running for 1.734)
2023-04-13T12:06:42.962Z  INFO 16276 --- [           main] o.s.b.a.b.JobLauncherApplicationRunner   : Running default command line with: []
2023-04-13T12:06:42.968Z  INFO 16276 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2023-04-13T12:06:42.977Z  INFO 16276 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```
