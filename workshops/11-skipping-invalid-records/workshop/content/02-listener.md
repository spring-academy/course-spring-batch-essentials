First, let's implement a `SkipListener` that writes skipped lines to a given file.

1. Create the `BillingDataSkipListener`.

   Open the `src/main/java/example/billingjob` folder and create a new file named `BillingDataSkipListener.java` with the following content:

   ```editor:append-lines-to-file
   file: ~/exercises/src/main/java/example/billingjob/BillingDataSkipListener.java
   description: "Create BillingDataSkipListener.java"
   ```

   ```java
   package example.billingjob;

   import java.io.IOException;
   import java.nio.file.Files;
   import java.nio.file.Path;
   import java.nio.file.Paths;
   import java.nio.file.StandardOpenOption;

   import org.springframework.batch.core.SkipListener;
   import org.springframework.batch.item.file.FlatFileParseException;

   public class BillingDataSkipListener implements SkipListener<BillingData, BillingData> {

   	Path skippedItemsFile;

   	public BillingDataSkipListener(String skippedItemsFile) {
   		this.skippedItemsFile = Paths.get(skippedItemsFile);
   	}

   	@Override
   	public void onSkipInRead(Throwable throwable) {
   		if (throwable instanceof FlatFileParseException exception) {
   			String rawLine = exception.getInput();
   			int lineNumber = exception.getLineNumber();
   			String skippedLine = lineNumber + "|" + rawLine + System.lineSeparator();
   			try {
   				Files.writeString(this.skippedItemsFile, skippedLine, StandardOpenOption.APPEND, StandardOpenOption.CREATE);
   			} catch (IOException e) {
   				throw new RuntimeException("Unable to write skipped item " + skippedLine);
   			}
   		}
   	}
   }
   ```

   That's a lot of code! Let's dive into it.

1. Understand the `BillingDataSkipListener`.

   Let's highlight some of the most important aspects of the `BillingDataSkipListener`.

   - This class implements the `SkipListener` interface.
   - The generic types, `<BillingData,BillingData>`, correspond to the type of input and output items of the step in which this listener will be registered, which is the `fileIngestion` step in our case.

     Remember, _this step does not change the item type_, `BillingData` - hence the notation, `<BillingData,BillingData>`.

   - The `BillingDataSkipListener` requires the path to the file in which skipped items should be written.

     Here, we pass the path to that file to the listener at construction time: `skippedItemsFile`. The path will be specified as a job parameter later when we start the job.

   - Finally, we implement the `onSkipInRead` method. This method will be invoked according to the skip policy, which we will define in the step in the next section.

     The exception we are interested in is `FlatFileParseException`. This exception gives a reference to the raw line that was read from the file as well as the line number.

     In our implementation, we take those two details from the exception and append them to the skip file according to the Pipe Separated Values specification.

That's all for the implementation of the `SkipListener`. Next up: the bean.
