#翻译 | 在 java 中处理临时文件和文件夹

> 原文自国外技术社区dzone，作者为 Anghel Leonard，[传送门](https://dzone.com/articles/working-with-temporary-filesfolders-in-java)

Java NIO.2 API 提供了对临时文件夹和文件的支持。例如，我们可以用下面来很轻松地定位到临时文件夹和文件的默认位置：

``` java
String defaultBaseDir = System.getProperty("java.io.tmpdir");
```

通常，在 Windows 系统中，默认的临时文件夹是 `C:\Temp` ， `%Windows%\Temp` 或者对于每个用户在 `Local Settings\Temp` 下的临时目录。（这个位置通常由 `TEMP` 环境变量控制）

在 Linux 或者 Unix 下，全局的临时目录是 `/tmp` 和 `/var/tmp`。上面的代码会根据操作系统返回默认的位置。下一步，我们将会学习如何创建一个临时文件夹及文件。

## 创建一个临时文件夹及文件

创建临时文件夹可以用这种方法来完成：

- `Path createTempDirectory (Path dir, String prefix, FileAttribute<?>... attrs)`

这是一个在 `Files` 类中的静态方法，可以按下面方式使用：

- 在当前操作系统的默认位置中创建一个不带前缀的临时文件夹：

  ``` java
  //C:\Users\Anghel\AppData\Local\Temp\8083202661590940905
  Path tmpNoPrefix = Files.createTempDirectory(null);
  ```

- 在当前操作系统的默认位置中创建一个带前缀的临时文件夹：

  ``` java
  // D:\tmp\logs_10153083118282372419
  Path customBaseDir = FileSystems.getDefault().getPath("D:/tmp");
  String customDirPrefix = "logs_";
  Path tmpCustomLocationAndPrefix = Files.createTempDirectory(customBaseDir, customDirPrefix);
  ```

创建临时文件可以用这种方法来完成：

- `Path createTempFile (Path dir, String prefix, String suffix, FileAttribute... attrs`

这是一个在 `Files` 类中的静态方法，可以按下面方式使用：

- 在当前操作系统的默认位置中创建一个不带前缀和后缀的临时文件：

  ```java
  // C:\Users\Anghel\AppData\Local\Temp\16106384687161465188.tmp
  2
  Path tmpNoPrefixSuffix = Files.createTempFile(null, null);
  ```

- 在当前操作系统的默认位置中创建一个带自定义前缀和后缀的临时文件：

  ``` java
  //C:\Users\Anghel\AppData\Local\Temp\log_402507375350226.txt
  String customFilePrefix = "log_";
  String customFileSuffix = ".txt";
  Path tmpCustomPrefixAndSuffix = Files.createTempFile(customFilePrefix, customFileSuffix);
  ```

- 在自定义位置中创建一个带自定义前缀和后缀的临时文件：

  ``` java
  //D:\tmp\log_13299365648984256372.txt
  Path customBaseDir = FileSystems.getDefault().getPath("D:/tmp");
  String customFilePrefix = "log_";
  String customFileSuffix = ".txt";
  Path tmpCustomLocationPrefixSuffix = Files.createTempFile(customBaseDir, customFilePrefix, customFileSuffix);
  ```

下一步，我们来看如何用不同的方式来删除临时文件夹和文件。

## 通过 Shutdown-hook 来删除临时文件夹和文件

我们可以通过操作系统或者特定的工具来删除临时文件夹和文件。然而，有时候，在某些不同的设计考虑中，我们需要使用程序式的方法来处理。

解决这个问题的一个方法是使用 shutdown-hook 机制，可以通过 `Runtime.getRuntime().addShutdownHook()` 方法来实现。每当我们需要在 JVM 关闭之前完成某些任务（例如清理任务）的时候，这个机制非常有用。它被实现为 Java 的一个线程，当 JVM 在关闭时执行 shutdown-hook 的时候，将会调用 `run()` 方法。就像下面的代码：

``` java
Path customBaseDir = FileSystems.getDefault().getPath("D:/tmp");
String customDirPrefix = "logs_";
String customFilePrefix = "log_";
String customFileSuffix = ".txt";
    
try {
  Path tmpDir = Files.createTempDirectory(customBaseDir, customDirPrefix);
  Path tmpFile1 = Files.createTempFile(tmpDir, customFilePrefix, customFileSuffix);
  Path tmpFile2 = Files.createTempFile(tmpDir, customFilePrefix, customFileSuffix);
  
  Runtime.getRuntime().addShutdownHook(new Thread() {
    
    @Override
    public void run() {
      try (DirectoryStream<Path> ds = Files.newDirectoryStream(tmpDir)) {
        for (Path file: ds) {
          Files.delete(file);
        }
        
        Files.delete(tmpDir);
      } catch (IOException e) {
        ...
      }
    }
  });
  
  //simulate some operations with temp file until delete it
  Thread.sleep(10000);
} catch (IOException | InterruptedException e) {
  ...
}
```

> 在异常或者强制终止（例如，JVM崩溃、终端操作被触发）的情况下，shutdown-hook 不会被执行。当所有的线程结束或者当 `System.exit(0)` 被调用的时候才会执行。建议尽快地执行，因为如果在出现问题（例如操作系统关闭），它们会被强制地终止。在编程的角度上，shutdown-hook 只能通过 `Runtime.halt()` 来终止。

##通过 deleteOnExit() 来删除临时文件夹和文件

另一个删除临时文件夹和文件的解决方法是通过 `File.deleteOnExit()` 方法。通过调用这个方法，我们能够注册这个删除动作，这个删除动作会在 JVM 停止的时候被执行：

``` java
Path customBaseDir = FileSystems.getDefault().getPath("D:/tmp");
String customDirPrefix = "logs_";
String customFilePrefix = "log_";
String customFileSuffix = ".txt";
    
try {
  Path tmpDir = Files.createTempDirectory(customBaseDir, customDirPrefix);
  System.out.println("Created temp folder as: " + tmpDir);
  Path tmpFile1 = Files.createTempFile(tmpDir, customFilePrefix, customFileSuffix);
  Path tmpFile2 = Files.createTempFile(tmpDir, customFilePrefix, customFileSuffix);
  
  try (DirectoryStream<Path> ds = Files.newDirectoryStream(tmpDir)) {
    tmpDir.toFile().deleteOnExit();
    
    for (Path file: ds) {      
      file.toFile().deleteOnExit();
    }
  } catch (IOException e) {
    ...
  }
  
  // simulate some operations with temp file until delete it
  Thread.sleep(10000);
} catch (IOException | InterruptedException e) {
  ...
}
```

> 建议在只有少量的临时文件夹和文件的时候才使用这个方法（`deleteOnExit()`）。这个方法可能会消耗大量内存（它会为每一个需要删除的临时资源任务分配内存），并且到 JVM 结束时才会释放这个内存。

##通过 DELETE_ON_CLOSE 来删除临时文件夹和文件

另一个删除临时资源的解决方法是通过 `StandardOpenOption.DELETE_ON_CLOSE`（当流结束时会删除文件）例如，如下的代码在通过 `createTempFile()` 方法创建临时文件之后，通过显式指定的 `DELETE_ON_CLOSE` 来开启一个 buffered writer 流：

``` java
Path customBaseDir = FileSystems.getDefault().getPath("D:/tmp");
String customFilePrefix = "log_";
String customFileSuffix = ".txt";
Path tmpFile = null;

try {
  tmpFile = Files.createTempFile(
    customBaseDir, customFilePrefix, customFileSuffix);
} catch (IOException e) {
  ...
}
     
try (BufferedWriter bw = Files.newBufferedWriter(tmpFile,
       StandardCharsets.UTF_8, StandardOpenOption.DELETE_ON_CLOSE)) {
  
  //simulate some operations with temp file until delete it
  Thread.sleep(10000);
} catch (IOException | InterruptedException e) {
  ...
}
```

> 这个解决方法适用于任何文件。它不局限于临时资源

