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

```c#
private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
{
    string rootResxDir = @"E:\Codes\Private\Syncfusion\WpfControlsLocalizationResxFiles\";
    DirectoryInfo resxDirInfo = new DirectoryInfo(rootResxDir);
    foreach (var resxDir in resxDirInfo.GetDirectories())
    {
        if (!resxDir.Name.StartsWith("Syncfusion."))
        {
            continue;
        }
        //Syncfusion.Chart.WPF
        //资源文件目录名称
        var resxDirName = resxDir.Name;
        //原始资源文件路径
        var sourceFilePath = Path.Combine(resxDir.FullName,resxDirName+".resx");
        //本地化资源文件路径
        var localizationFilePath = Path.Combine(resxDir.FullName,resxDirName+".zh-CN.resx");
        //本地化资源文件路径是否存在、原始资源文件路径是否存在
        if (!File.Exists(sourceFilePath))
        {
            Debug.WriteLine($"原始文件： {sourceFilePath} 不存在，请查证。");
           continue;
        }
        if (!File.Exists(localizationFilePath))
        {
            Debug.WriteLine($"本地化文件： {localizationFilePath} 不存在，请查证。");
            continue;
        }
        // 本地化资源文件备份路径
        var localizationBackFilePath = Path.Combine(resxDir.FullName,resxDirName+".back.zh-CN.resx");
        // 备份文件不存在进行一次备份。
        if (!File.Exists(localizationBackFilePath))
        {
            //File.Copy(localizationFilePath, localizationBackFilePath,true);
        }

        //读取原始文件Xml
        XmlDocument docSource = new XmlDocument
        {
            PreserveWhitespace = true
        };
        docSource.Load(sourceFilePath);

        //读取本地化文件Xml
        XmlDocument docLocalization = new XmlDocument
        {
            PreserveWhitespace = true
        };
        docLocalization.Load(localizationFilePath);

        // 定位根节点
        XmlNode? rootLocalization = docLocalization.SelectSingleNode("//root");
        XmlNode? lastDataLocalization = docLocalization.SelectSingleNode("//root/data[last()]");

        // 遍历文件 A 的 data 节点
        XmlNodeList? dataNodesSource = docSource.SelectNodes("//root/data");
        if (dataNodesSource == null || rootLocalization == null || lastDataLocalization == null)
        {
            continue;
        }
        bool isChange  = false;
        foreach (XmlNode dataNodeSource in dataNodesSource)
        {
            if (dataNodesSource == null)
            {
                continue;
            }
            string? name = dataNodeSource!.Attributes!["name"]!.Value;
            XmlNode? existingNode = docLocalization.SelectSingleNode($"//root/data[@name='{name}']");
            if (existingNode == null)
            {
                // 导入并插入节点
                XmlNode importedNode = docLocalization.ImportNode(dataNodeSource, true);
                rootLocalization.InsertAfter(importedNode, lastDataLocalization);
                lastDataLocalization = importedNode;
                isChange = true;
            }
        }

        if (isChange)
        {
            // 保存文件
            XmlWriterSettings settings = new XmlWriterSettings
            {
                Indent = true,
                IndentChars = "  ",
                Encoding = new System.Text.UTF8Encoding(false),
                NewLineChars = "\n"
            };
            using XmlWriter writer = XmlWriter.Create(localizationFilePath, settings);
            docLocalization.Save(writer);
        }
        // CompareAndMergeNodes(docSource.DocumentElement, docLocalization.DocumentElement,docLocalization);
        // docLocalization.Save(localizationFilePath);
    }
}

/// <summary>
/// 比较和合并节点
/// </summary>
/// <param name="parentNodeSource"></param>
/// <param name="parentNodeTarget"></param>
/// <param name="docTarget"></param>
public void CompareAndMergeNodes(XmlNode? parentNodeSource, XmlNode? parentNodeTarget,XmlDocument docTarget)
{
    if (parentNodeSource == null)
    {
        return;
    }

    parentNodeTarget ??= new XmlDocument();

    foreach (XmlNode nodeSource in parentNodeSource.ChildNodes)
    {
        // 跳过注释节点
        if (nodeSource.NodeType == XmlNodeType.Comment)
        {
            continue;
        }

        // 查找旧文件中是否存在相同节点
        bool exists = false;
        foreach (XmlNode nodeTarget in parentNodeTarget.ChildNodes)
        {
            if (IsSameNode(nodeSource, nodeTarget))
            {
                exists = true;
                CompareAndMergeNodes(nodeSource, nodeTarget, docTarget); // 递归比较子节点
                break;
            }
        }

        // 若不存在，则导入并插入新节点
        if (!exists)
        {
            XmlNode importedNode = docTarget.ImportNode(nodeSource, true);
            parentNodeTarget.AppendChild(importedNode);
        }
    }
}

// 判断旧文件中是否已存在相同内容的注释
public bool ContainsComment(XmlNode parent, string commentText)
{
    foreach (XmlNode node in parent.ChildNodes)
    {
        if (node.NodeType == XmlNodeType.Comment 
            && node.Value == commentText)
        {
            return true;
        }
    }
    return false;
}

/// <summary>
/// 判断两个节点是否相同（标签名、属性、内容）
/// </summary>
/// <param name="nodeSource"></param>
/// <param name="nodeTarget"></param>
/// <returns></returns>
public bool IsSameNode(XmlNode nodeSource, XmlNode nodeTarget)
{
    return nodeSource.Name == nodeTarget.Name 
           && nodeSource.InnerXml == nodeTarget.InnerXml 
           && CompareAttributes(nodeSource.Attributes, nodeTarget.Attributes);
}

/// <summary>
/// 比较节点属性是否一致
/// </summary>
/// <param name="attrsSource"></param>
/// <param name="attrsTarget"></param>
/// <returns></returns>
bool CompareAttributes(XmlAttributeCollection? attrsSource, XmlAttributeCollection? attrsTarget)
{
    if (attrsSource == null)
    {
        if (attrsTarget != null)
        {
            return false;
        }
        else
        {
            return true;
        }
    }
    if (attrsTarget == null)
    {
        return false;
    }
    if (attrsSource?.Count != attrsTarget?.Count) return false;
    foreach (XmlAttribute attr in attrsSource!)
    {
        if (attrsTarget![attr.Name]?.Value != attr.Value) return false;
    }
    return true;
}
```

