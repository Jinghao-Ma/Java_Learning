# Maven





[TOC]



### 简介



- Maven是一个Java项目管理和构建的工具
- Maven使用`pom.xml`定义项目内容，并使用预设的目录结构
- 在Maven中声明一个依赖项可以自动下载并导入`classpath`
- ==**Maven使用`groupId`, `artifactId`和`version`定位唯一一个jar包**==

```xml
<!-- pom.xml -->
<project>
	<modelVersion>4.0.0</modelVersion>
    
    <groupId>org.maventest.mavendemo</groupId>
    <artifactId>MavenTest</artifactId>
    <version>1.0-SNAPSHOT</version>
</project>
```

------



### 依赖管理

#### Maven的依赖关系

| scope           | 说明                                          | 示例            |
| --------------- | --------------------------------------------- | --------------- |
| ==**compile**== | 编译时需要用到该jar包                         | commons-logging |
| test            | 编译Test时需要用到该jar包                     | junit           |
| runtime         | 编译时不需要，但运行时需要用到                | log4j           |
| provided        | 编译时需要用到，但运行时由JDK或某个服务器提供 | servlet-api     |



**Maven下载所需依赖**

- Maven维护了一个中央仓库（**repo1.maven.org**）
- 第三方库将自身上传至中央仓库
- Maven从中央仓库把所需依赖下载到本地
- Maven会自动缓存已下载过的jar包（**~/.m2/repository/**）



#### Maven镜像配置

**可以在~/.m2下的settings.xml创建自己的镜像仓库 **

```xml
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>aliyun</name>
            <mirrorOf>central</mirrorOf>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>
    </mirrors>
</settings>
```

------



### 构建流程

**Maven是一个Java项目管理和构建工具**

- 标准化项目结构
- 标准化构建流程（编译，打包，发布）
- 依赖管理



**Maven构建流程：**

1. `clean`: 删除所有编译生成的文件
2. `compile`： 编译源码，编译测试源码
3. `test`：运行测试
4. `package`：打包为jar



**Maven的生命周期（lifecycle）由一系列阶段（Phase）构成：**

- `clean`
- `compile`
- `test`



```bash
# 对maven项目执行compile
mvn compile

# 对maven项目执行clean、compile和test
mvn clean test

# 对maven项目执行package（打jar包）
mvn clean package
```



**Lifecycle、Phase、Goal**

- 使用Maven构建项目就是执行**Lifecycle**
- 执行**Lifecycle**就是按顺序执行一系列**Phase**
- 每执行一个**Phase**都会执行该**Phase**绑定的**Goal**
- **Goal**是最小执行任务单元

**常用命令：==mvn clean package (不会打包有依赖的jar)==**

------



### Maven插件

Maven通过调用不同的插件（**plugin**）来构建项目

- mvn compile
  - 调用compiler插件执行compile
  - compiler插件执行和compile关联的`compiler:complie`这个**Goal**



#### Maven的常用标准插件

| 插件名称 | 对应执行的Phase |
| -------- | --------------- |
| clean    | `clean`         |
| compiler | `compile`       |
| surefire | `test`          |
| jar      | `package`       |

- maven-shade-plugin: 打包所有依赖包并生成可执行jar
- cobertura-maven-plugin：生成单元测试覆盖率报告
- findbugs-maven-plugin：对Java源码进行静态分析以找出潜在问题



#### maven-shade-plugin配置

pom.xml

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.2</version>
    <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass><!-- 指定主类路径 如：org.maventest.Main--></mainClass>
            </transformer>
        </transformers>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

执行`mvn clean package`后：

如果出现==**jar中没有主清单属性**==的告警时，可以修改**pom.xml**中的jar配置

```xml
<plugin>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass><!-- 指定主类路径 如：org.maventest.Main--></mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin
```



#### cobertura-maven-plugin配置

pom.xml

```xml
<dependency>
    <groupId>net.sourceforge.cobertura</groupId>
    <artifactId>cobertura</artifactId>
    <version>2.1.1</version>
    <scope>test</scope>
</dependency>
```

执行命令`mvn cobertura:cobertura`

`MavenDemo\target\site\cobertura\index.html` 中可以查看单元测试覆盖率报告



- Maven通过自定义插件可以执行项目构建时需要的额外功能
- 在pom.xml中声明插件及配置
- 插件会在某个Phase被执行
- 插件的配置和用法需参考插件官方文档

 

------



### 模块管理



**把大项目拆分为模块是降低软件复杂度的有效方法：**

- $Single Project \longrightarrow  \left\{\begin{aligned} Module_A \\ Module_B \\ Module_C \\ ... \end{aligned}\right.$

**Maven可以有效管理多个模块**

**如果模块A和模块B的`pom.xml`高度相似，可以提取出沟通的部分作为parent** 

==**Maven Modules parent 只需配置pom.xml （可删除所有代码文件）**==

```xml
<project>  
  <groupId>org.maventest.mavendemo</groupId>
  <artifactId>MavenParent</artifactId>
  <version>1.0-SNAPSHOT</version>
  <!--该packaging必须声明为pom-->
  <packaging>pom</packaging>
  
  <!-- 相关属性 -->
  <properties>
      ...
  </properties>
    
  <!-- 依赖项 -->
  <dependencies>
      <dependency>...</dependency>
      ...
  </dependencies>
  
  <!-- 插件 -->
  <build>
      <pluginManagement>
          <plugin>...</plugin>
          ...
      </pluginManagement>
  </build>
  
</project>
```



**在各个Module中套用parent**

```xml
<!--引用parent模块-->
<parent>
    <groupId>org.maventest.mavendemo</groupId>
    <artifactId>MavenParent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../MavenPrjParent/pom.xml</relativePath>
</parent>

<!--该模块自身的信息-->
<artifactId>...</artifactId>
<version>...</version>

<!--该模块自有的依赖或插件-->
<dependencies>
      <dependency>...</dependency>
      ...
</dependencies>
  
<build>
      <pluginManagement>
          <plugin>...</plugin>
          ...
      </pluginManagement>
</build>
```



**如果模块A依赖模块B，则模块A需要模块B的jar包才能正常编译**：

- 中央仓库
- 本地仓库（`~/.m2`）
- 模块化编译



==**模块化编译 ： build模块只需配置pom.xml（可删除所有代码文件）**==

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.maventest.mavendemo</groupId>
    <artifactId>MavenParent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <!--
        通过modules->module来进行多模块编译
        如有parent引用，必须也加上parent模块
    -->
    <modules>
        <module>../MavenPrjParent</module>
        <module>../MavenPrjFirst</module>
        <module>../MavenPrjSecond</module>
    </modules>
</project>
```

