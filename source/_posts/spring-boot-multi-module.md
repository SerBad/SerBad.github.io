---
title: Spirng boot maven多模块打包踩坑
tags: spring boot
comments: false
date: 2017-11-02 10:42:28
---
最近折腾了两次spring boot在maven下的多模块打包，踩了很多坑，现在记录如下。
<!-- more -->
项目目录：
- 项目 P
- 模块 A
- 模块 B
- 公有基础模块 C
- Mybatis基础模块 M

# 父pom.xml文件：
```xml
  <!--版本号-->
  <groupId>com.parent</groupId>
  <artifactId>demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <!--管理所有的模块-->
  <modules>
    <modules>C</modules>
    <modules>M</modules>
  </modules>

  <!--指定项目的spring boot的版本-->
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.0.0.M5</version>
  </parent>

  <!--指定项目中公有的模块-->
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.module</groupId>
        <artifactId>c</artifactId>
        <version>${project.version}</version>
      </dependency>
      <dependency>
        <groupId>com.module</groupId>
        <artifactId>m</artifactId>
        <version>${project.version}</version>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <!--指定jdk的版本为1.8，默认为1.6-->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <!--指定项目中公有的依赖-->
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
  </dependencies>

  <!--指定使用maven打包-->
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
          <configuration>
            <source>${java.version}</source>
            <target>${java.version}</target>
            </configuration>
      </plugin>

      <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.19.1</version>
            <configuration>
              <skipTests>true</skipTests>    <!--默认关掉单元测试 -->
            </configuration>
      </plugin>
    </plugins>
  </build>
```
# 模块A的pom.xml
```xml
  <groupId>com.module</groupId>
  <artifactId>a</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <!--指定父模块，需要注意的是，这里要指定父模块pom.xml的相对路径-->
  <parent>
    <groupId>com.parent</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>

  <!--spring boot打包的话需要指定一个唯一的入门-->
  <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
          <configuration>
            <!-- 指定该Main Class为全局的唯一入口 -->
            <mainClass>com.module.a.Application</mainClass>
            <layout>ZIP</layout>
          </configuration>
          <executions>
            <execution>
              <goals>
                <goal>repackage</goal><!--可以把依赖的包都打包到生成的Jar包中-->
              </goals>
            </execution>
          </executions>
      </plugin>
		</plugins>
	</build>
```
# 模块B的pom.xml
同A即可

# 模块C的pom.xml
如果是共有模块的话，不需要打包，否则会报错，因为其他模块在打包的时候会自动添加依赖进去，如果这里打包了，其他的模块就找不到该依赖了。
```xml
  <groupId>com.module</groupId>
	<artifactId>c</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

  <!--指定父模块，需要注意的是，这里要指定父模块pom.xml的相对路径-->
  <parent>
    <groupId>com.parent</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
```

# 模块M的pom.xml
如果项目中使用的Mybatis的话，肯定是作为一个单独的模块来处理的，这个Mybatis是需要打包的
```xml
  <groupId>com.module</groupId>
  <artifactId>m</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <!--指定父模块，需要注意的是，这里要指定父模块pom.xml的相对路径-->
  <parent>
    <groupId>com.parent</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>

  <!--mybatis的打包方式-->
  <build>
		<plugins>
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.5</version>
				<executions>
					<execution>
						<id>Generate MyBatis Artifacts</id>
						<phase>none</phase>
						<goals>
							<goal>generate</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<overwrite>true</overwrite>
				</configuration>
			</plugin>
		</plugins>
	</build>
```
# 打包
按照上面的配置好以后，执行下面的命令就好了
```
  mvn clean package
```

但是如果使用了多个模块，上面的命令是会吧全部的模块都执行打包的，如果只是打包某个模块的话，可以用
```
  mvn -pl A -am install
```
