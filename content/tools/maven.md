---
title: "maven"
date: 2018-01-22 22:53
---

## Eclipse Maven 编译错误：java: try-with-resources is not supported in -source 1.5 (use -source 7 or higher to enable try-with-resources)
要彻底解决这个问题只需在pom文件中配置maven-compiler-plugin并指定编译器的版本为你想要的版本即可：

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```