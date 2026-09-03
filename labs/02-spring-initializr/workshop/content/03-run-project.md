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

 :: Spring Boot ::                (v4.1.1)

2026-09-03T07:50:30.603Z  INFO 1502 --- [Billing Job] [           main] e.billingjob.BillingJobApplication       : Starting BillingJobApplication using Java 21.0.8 with PID 1502 (/home/eduk8s/exercises/billing-job/target/classes started by eduk8s in /home/eduk8s/exercises)
2026-09-03T07:50:30.607Z  INFO 1502 --- [Billing Job] [           main] e.billingjob.BillingJobApplication       : No active profile set, falling back to 1 default profile: "default"
2026-09-03T07:50:32.228Z  INFO 1502 --- [Billing Job] [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2026-09-03T07:50:32.571Z  INFO 1502 --- [Billing Job] [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@46a953cf
2026-09-03T07:50:32.574Z  INFO 1502 --- [Billing Job] [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2026-09-03T07:50:32.777Z  INFO 1502 --- [Billing Job] [           main] e.billingjob.BillingJobApplication       : Started BillingJobApplication in 2.829 seconds (process running for 3.447)
2026-09-03T07:50:32.782Z  INFO 1502 --- [Billing Job] [           main] o.s.b.b.a.JobLauncherApplicationRunner   : Running default command line with: []
2026-09-03T07:50:32.790Z  INFO 1502 --- [Billing Job] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2026-09-03T07:50:32.811Z  INFO 1502 --- [Billing Job] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```
