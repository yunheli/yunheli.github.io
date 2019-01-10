
---
layout: post
title: use github as maven repository
date: 2018-05-05 00:00
tags: [github, repository]
---


#### 简介
下面我们讲述如何使用Github来作为maven仓库，公开仓库简单，重点私有仓库如何做

#### 首先创建一个maven项目，下面是pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.anupam</groupId>
    <artifactId>test-repo</artifactId>
    <version>1.0.0</version>
    <build>
        <plugins>
	      <plugin>
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-compiler-plugin</artifactId>
	        <configuration>
	          <source>8</source>
	          <target>8</target>
	        </configuration>
	      </plugin>

	      <plugin>
	        <groupId>org.jacoco</groupId>
	        <artifactId>jacoco-maven-plugin</artifactId>
	        <version>0.8.1</version>
	        <executions>
	          <execution>
	            <id>default-prepare-agent</id>
	            <goals>
	              <goal>prepare-agent</goal>
	            </goals>
	          </execution>
	          <execution>
	            <id>default-report</id>
	            <goals>
	              <goal>report</goal>
	            </goals>
	          </execution>
	          <execution>
	            <id>post-unit-test</id>
	            <phase>test</phase>
	            <goals>
	              <goal>report</goal>
	            </goals>
	            <configuration>
	              <dataFile>target/jacoco.exec</dataFile>
	              <outputDirectory>target/jacoco-ut</outputDirectory>
	            </configuration>
	          </execution>
	        </executions>
	      </plugin>

	      <plugin>
	        <groupId>com.github.github</groupId>
	        <artifactId>site-maven-plugin</artifactId>
	        <version>0.11</version>
	        <configuration>
	          <message>Maven artifacts for ${project.version}</message>
	          <noJekyll>true</noJekyll>
	          <outputDirectory>${project.build.directory}/mvn-repo
	          </outputDirectory>
	          <merge>true</merge>
	          <branch>refs/heads/mvn-repo</branch>
	          <includes>
	            <include>**/*</include>
	          </includes>
	          <repositoryName>akey-common-util</repositoryName>
	          <repositoryOwner>yunheli</repositoryOwner>
	        </configuration>
	        <executions>
	          <execution>
	            <goals>
	              <goal>site</goal>
	            </goals>
	            <phase>deploy</phase>
	          </execution>
	        </executions>
	      </plugin>
	    </plugins>
    </build>
    <distributionManagement>
        <repository>
            <id>internal</id>
            <url>file://${project.build.directory}/mvn-repo</url>
        </repository>
    </distributionManagement>
</project>
```

最重要的是下面这段
```xml
<distributionManagement>
    <repository>
        <id>internal</id>
        <url>file://${project.build.directory}/mvn-repo</url>
    </repository>
</distributionManagement>
```

之后执行deploy
```sh
mvn clean deploy
```

#### 其他项目如何引用呢？

- 公开项目怎么做？
	
在pom里面加下面就行了
```xml
	<repositories>
	    <repository>
	      <id>github-domain</id>
	      <url>https://raw.github.com/yunheli/akey-common-domain/mvn-repo/</url>
	      <snapshots>
	        <enabled>true</enabled>
	        <updatePolicy>always</updatePolicy>
	      </snapshots>
	    </repository>

	    <repository>
	      <id>github-util</id>
	      <url>https://raw.github.com/yunheli/akey-common-util/mvn-repo/</url>
	      <snapshots>
	        <enabled>true</enabled>
	        <updatePolicy>always</updatePolicy>
	      </snapshots>
	    </repository>
  </repositories>
```

	之后在~/.m2/setting.xml里面配置server

```xml
	<server>
      <id>github-domain</id>
      <username>yunheli</username>

      <!-- 公开仓库就不需要configuration了-->
      <configuration>
        <httpHeaders>
          <property>
            <name>Authorization</name>
            <!-- 私有仓库需要加上下面一行-->
            <!-- Base64-encoded username:access_token -->
            <value>Basic xxxxxx==</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>

    <server>
      <id>github-util</id>
      <username>yunheli</username>
      <configuration>
        <httpHeaders>
          <property>
            <name>Authorization</name>
            <!-- 私有仓库需要加上下面一行-->
            <!-- Base64-encoded username:access_token -->
            <value>Basic xxxxxx==</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>
```