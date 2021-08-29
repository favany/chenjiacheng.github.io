---
layout: post
categories: PHP
tags: [PHP,ThinkPHP]
---

在 IIS 的高版本下面可以配置 web.config，在中间添加 rewrite 节点

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>  
            <rules>  
                <rule name="WPurls" enabled="true" stopProcessing="true">  
                    <match url=".*" />  
                    <conditions logicalGrouping="MatchAll">  
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />  
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />  
                    </conditions>  
                    <action type="Rewrite" url="index.php/{R:0}" />  
                </rule>  
            </rules>  
        </rewrite>

        <defaultDocument>
            <files>
                <clear />
                <add value="index.php" />
                <add value="index.html" />
            </files>
        </defaultDocument>
    </system.webServer>
</configuration>
```