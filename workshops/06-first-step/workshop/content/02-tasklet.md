Create a new file named `FilePreparationTasklet.java` in the `src/main/java/example/billingjob` folder and update its content as follows:

```editor:append-lines-to-file
file: ~/exercises/src/main/java/example/billingjob/FilePreparationTasklet.java
description: "Create FilePreparationTasklet.java"
```

```java
package example.billingjob;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

public class FilePreparationTasklet implements Tasklet {

	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
		JobParameters jobParameters = contribution.getStepExecution().getJobParameters();
		String inputFile = jobParameters.getString("input.file");
		Path source = Paths.get(inputFile);
		Path target = Paths.get("staging", source.toFile().getName());
		Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
		return RepeatStatus.FINISHED;
	}
}
```

In this class, we implement the `Tasklet` interface to get the input file from job parameters and copy it in the staging directory.

Note how we used the `StandardCopyOption.REPLACE_EXISTING` option when copying the file in order to replace any existing file if any. This is useful in case the step is re-executed and we want it to succeed instead of failing because the file already exists in the directory.

With that in place, we are now ready to define the `TaskletStep`.
