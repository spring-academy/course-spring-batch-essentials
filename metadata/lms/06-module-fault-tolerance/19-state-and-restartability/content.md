Batch jobs are never executed in isolation. They consume data, produce data and often interact with external components and services. This interaction with the external world makes them exposed to different kinds of human errors and system faults like receiving bad input data or interacting with unreliable services.

As a batch developer, you're faced with several challenges: How should you address incorrect input? How should you handle processing errors? These and other questions are all real-world problems that almost all batch systems encounter.

Therefore, you should design and implement your applications in a robust and fault-tolerant way, and Spring Batch can help you tremendously in that task.

In this lesson, we'll introduce the fault-tolerance features in Spring Batch and explain when to use them.

## State Management

We've already seen how the domain model of Spring Batch is designed to allow the restart of failed job instances. In fact, thanks to saving the state of each job execution in a persistent job repository, Spring Batch is able to resume job instances where they left off.

Restartability in Spring Batch is implemented at two distinct levels: _inter-step_ restartability and _inner-step_ restartability.

### Inter-Step Restartability

Inter-step restartability refers to _resuming a job from the last failed step_, without re-executing the steps that were successfully executed in the previous run.

For instance, if a job is composed of 2 sequential steps and fails at the second step, then the first step won't be re-executed in case of a restart.

### Inner-Step Restartability

Inner-step restartability refers to _resuming a failed step where it left off_, i.e. from within the step itself.

This feature isn't particular to a specific type of steps, but is typically related to chunk-oriented steps. In fact, the chunk-oriented processing model is designed to be tolerant to faults and to restart from the last save point, meaning that successfully processed chunks aren't re-processed again, in the case of a restart.

We'll explore this with an example in the next Lab.

## Restarting Failed Jobs

Restartability is a feature in Spring Batch that you don't activate with a flag or by using a specific API. Rather, it's a feature that's provided automatically by the framework, thanks to its powerful domain model design. In fact, restarting a failed job instance is just a matter of relaunching it with the same identifying job parameters. There are no APIs to call or features to activate.

## Error Handling

By default, if an exception occurs in a given step, the step and its enclosing job will fail. At this point, you'll have several choices about how to proceed, depending on the nature of the error:

- If the error is _transient_ (like a failed call to a flaky web service), you can decide to restart the job. The job will restart where it left off and might succeed the second time. However, this is not guaranteed as the transient error might happen again. In this case, you'd want to find a way to implement a retry policy around the operation that might fail.
- If the error is _not transient_ (like an incorrect input data), then restarting the job won't solve the problem. In this case, you'll need to decide if you want to either fix the problem and restart the job, or to tolerate the bad input and skip it for later analysis or reprocessing.

For each of these situations, Spring Batch provides a specific feature: Retry and Skip, which we'll explain in detail in the next lessons.
