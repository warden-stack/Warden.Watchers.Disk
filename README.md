# Warden Disk Watcher

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Disk.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Watchers.Disk)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Disk.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Watchers.Disk/branches)

**DiskWatcher** can be used either for basic disk monitoring such as finding the specified partitions, directories or files and validating available disk space.

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Watchers.Disk)**. 
```
dotnet add package Warden.Watchers.Disk
```

### Configuration:

 - **WithPartitionsToCheck()** - one or more partitions to check e.g. *"C:\"*.
 - **WithDirectoriesToCheck()** - one or more directories to check e.g. *"C:\Files"*.
 - **WithFilesToCheck()** - one or more files to check e.g. *"C:\Files\Log.txt"*.
 - **EnsureThat()** - predicate containing a received query result of type *DiskCheck* that has to be met in order to return a valid result.
 - **EnsureThatAsync()** - async  - predicate containing a received query result of type *DiskCheck* that has to be met in order to return a valid result
 - **WithDiskCheckerProvider()** - provide a  custom *IDiskChecker* which is responsible for executing the disk check operation.

**DiskWatcher** can be configured by using the **DiskWatcherConfiguration** class or via the lambda expression passed to a specialized constructor. 


Example of configuring the watcher via provided configuration class:
```csharp
var configuration = DiskWatcherConfiguration
    .Create()
    .WithFilesToCheck(@"D:\Test\File1.txt", @"D:\Test\File2.txt")
    .WithPartitionsToCheck("D", @"E:\")
    .WithDirectoriesToCheck(@"D:\Test")
    .EnsureThat(check => check.FreeSpace > 100000000)
    .Build();
var diskWatcher = DiskWatcher.Create("My Disk watcher", configuration);

var wardenConfiguration = WardenConfiguration
    .Create()
    .AddWatcher(diskWatcher)
    //Configure other watchers, hooks etc.
```

Example of adding the watcher directly to the **Warden** via one of the extension methods:
```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .AddDiskWatcher(cfg =>
    {
        cfg.WithFilesToCheck(@"D:\Test\File1.txt", @"D:\Test\File2.txt")
           .WithPartitionsToCheck("D", @"E:\")
           .WithDirectoriesToCheck(@"D:\Test")
           .EnsureThat(check => check.FreeSpace > 100000000)
    })	
    //Configure other watchers, hooks etc.
```

Please note that you may either use the lambda expression for configuring the watcher or pass the configuration instance directly. You may also configure the **hooks** by using another lambda expression available in the extension methods.

### Check result type:
**DiskWatcher** provides a custom **DiskWatcherCheckResult** type which contains additional value.

```csharp
public class DiskWatcherCheckResult : WatcherCheckResult
{
    public DiskCheck DiskCheck { get; }
}
```

### Custom interfaces:
```csharp
public interface IDiskChecker
{
    Task<DiskCheck> CheckAsync(IEnumerable<string> partitions = null,
        IEnumerable<string> directories = null, IEnumerable<string> files = null);
}
```

**IDiskChecker** is responsible for making a connection to the database. It can be configured via the *WithDiskCheckerProvider()* method. By default it is based on the **System.IO**.


```csharp
public class DiskCheck
{
    public long FreeSpace { get; }
    public long UsedSpace { get; }
    public long TotalSpace => UsedSpace + FreeSpace;
    public IEnumerable<PartitionInfo> Partitions { get; }
    public IEnumerable<DirectoryInfo> Directories { get; }
    public IEnumerable<FileInfo> Files { get; }
}

public class PartitionInfo
{
    public string Name { get; }
    public long UsedSpace { get; }
    public long FreeSpace { get; }
    public long TotalSpace => UsedSpace + FreeSpace;
    public bool Exists { get; }
}

public class DirectoryInfo
{
    public string Name { get; }
    public string Path { get; }
    public int FilesCount { get; }
    public long Size { get; }
    public bool Exists { get; }
}

public class FileInfo
{
    public string Name { get; }
    public string Path { get; }
    public string Extension { get; }
    public bool Exists { get; }
    public long Size { get; }
    public string Partition { get; }
    public string Directory { get; }
}
```

**DiskCheck** is a custom type that holds the values related to performed watcher check.