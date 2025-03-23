# 中文本地化

大部分内容使用机翻。



## 旧版原始资源文件Copy代码片段

```csharp
public void CopyFile()
{
    string targetDir = @"E:\Codes\Private\Syncfusion\WpfControlsLocalizationResxFiles\";
    string sourceDir = @"E:\Codes\Private\Urp2024\urp-front-machine\src\framework\src\Shared\Localization.Syncfusion.WPF";
    DirectoryInfo targetDirInfo = new DirectoryInfo(targetDir);
    foreach (var targetDirItem in targetDirInfo.GetDirectories())
    {
        if (!targetDirItem.Name.StartsWith("Syncfusion."))
        {
            continue;
        }
        //Syncfusion.Chart.WPF
        var targetDirName = targetDirItem.Name;
        //Syncfusion.Chart.WPF.resx
        string sourceFileName = Path.Combine(sourceDir, targetDirName+".zh-CN.resx");
        //原始文件覆盖
        if (File.Exists(sourceFileName))
        {
            var targetFileName = Path.Combine(targetDir,targetDirName,targetDirName + ".zh-CN.resx");
            if (!File.Exists(targetFileName))
            {
                //File.Delete(targetFileName);
                File.Copy(sourceFileName, targetFileName, true);
            }
        }
    }
}
```

# 新版原始资源文件与旧版简中资源文件差异内容生成