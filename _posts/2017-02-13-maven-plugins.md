---
layout: post
title: 【Maven】-Maven插件
date: 2017-02-13 16:16:16
categories: [Maven]
tags: [maven,plugin]
baseline:
---

### 常用参数

- -Dmaven.test.skip=true 跳过测试
- -f 指定 pom.xml 文件

### Maven Lifecycle

- clean

用于清除之前构建生成的所有文件 target 目录

- validate

用于验证项目是否正确

- compile

编译项目源代码 主要是 java 文件

- test

测试编译出来的代码

- packaging

获取编译好的代码并将其打包成相应类型 jar war

- verify

用来验证 test 检查 test 的结果是否满足标准

- install

将软件包安装到本地存储

- deploy

复制最终的包到远程仓库

#### maven-compiler-plugin
```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<configuration>
		<compilerArguments>
			<extdirs>lib</extdirs>
		</compilerArguments>
		<forceJavacCompilerUse>true</forceJavacCompilerUse>
	</configuration>
</plugin>
```
#### maven-dependency-plugin

参考 https://maven.apache.org/plugins/maven-dependency-plugin/index.html

```
<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<version>2.7</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>copy-dependencies</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<includeScope>system</includeScope>
					<outputDirectory>${project.build.directory}/${project.build.finalName}/WEB-INF/lib</outputDirectory>
				</configuration>
			</plugin>
```

#### maven-war-plugin
```
<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.4</version>
				<configuration>
					<includeEmptyDirectories>true</includeEmptyDirectories>
					<webResources>
						<resource>
							<directory>../lib</directory>
							<targetPath>WEB-INF/lib</targetPath>
							<includes>
								<include>**/*.jar</include>
							</includes>
						</resource>
					</webResources>
				</configuration>
			</plugin>
```

#### Maven Deploy

```
<plugin>
  <groupId>com.github.wvengen</groupId>
  <artifactId>proguard-maven-plugin</artifactId>
  <version>2.0.7</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>proguard</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <attach>true</attach>
    <attachArtifactClassifier>pg</attachArtifactClassifier>
    <!-- attach 的作用是在 install 与 deploy 时将生成的 pg 文件也安装与部署 -->
    <options> <!-- 详细配置方式参考 ProGuard 官方文档 -->
      <!--<option>-dontobfuscate</option>-->
      <option>-ignorewarnings</option> <!--忽略所有告警-->
      <option>-dontshrink</option>   <!--不做 shrink -->
      <option>-dontoptimize</option> <!--不做 optimize -->
      <option>-dontskipnonpubliclibraryclasses</option>
      <option>-dontskipnonpubliclibraryclassmembers</option>

      <option>-repackageclasses org.noahx.proguard.example.project2.pg</option>
      <!--平行包结构（重构包层次），所有混淆的类放在 pg 包下-->

      <!-- 以下为 Keep，哪些内容保持不变，因为有一些内容混淆后（a,b,c）导致反射或按类名字符串相关的操作失效 -->

      <option>-keep class **.package-info</option>
      <!--保持包注解类-->

      <option>-keepattributes Signature</option>
      <!--JAXB NEED，具体原因不明，不加会导致 JAXB 出异常，如果不使用 JAXB 根据需要修改-->
      <!-- Jaxb requires generics to be available to perform xml parsing and without this option ProGuard was not retaining that information after obfuscation. That was causing the exception above. -->

      <option>-keepattributes SourceFile,LineNumberTable,*Annotation*</option>
      <!--保持源码名与行号（异常时有明确的栈信息），注解（默认会过滤掉所有注解，会影响框架的注解）-->

      <option>-keepclassmembers enum org.noahx.proguard.example.project2.** { *;}</option>
      <!--保持枚举中的名子，确保枚举 valueOf 可以使用-->

      <option>-keep class org.noahx.proguard.example.project2.bean.** { *;}</option>
      <!--保持 Bean 类，（由于很多框架会对 Bean 中的内容做反射处理，请根据自己的业务调整） -->

      <option>-keep class org.noahx.proguard.example.project2.Project2 { public void init(); public void
        destroy(); }
      </option>
      <!-- 保持对外的接口性质类对外的类名与方法名不变 -->

    </options>
    <outjar>${project.build.finalName}-pg</outjar>
    <libs>
      <lib>${java.home}/lib/rt.jar</lib>
    </libs>

  </configuration>
</plugin>
```

#### mybatis-generator-maven-plugin

```
<plugin>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-maven-plugin</artifactId>
	<version>1.3.2</version>
	<configuration>
		<configurationFile>src/main/resources/mybatis-generator.xml</configurationFile>
		<verbose>true</verbose>
		<overwrite>true</overwrite>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.3.2</version>
		</dependency>
	</dependencies>
</plugin>
```

> http://blog.csdn.net/woshixuye/article/details/8133050
