## 研发知识学习

​			优秀博主：

> https://www.jianshu.com/u/86696f09d988
>
> 



- **.gitignore文件是什么**

  ​	.gitignore文件是一个用于Git版本控制系统的特殊文件，用于指定Git版本控制系统中哪些文件和目录应该被忽略，以避免将不必要的文件添加到版本控制系统中。 

  ​	.gitignore文件中列出的文件和目录将不会被Git跟踪和提交。一般情况下，.gitignore文件会列出一些临时文件、日志文件、编译生成的文件、与系统环境相关的文件等等，确保这些文件不会被意外添加到版本库中，从而保持版本库的干净和安全。 

  ​	.gitignore文件通常位于Git仓库的根目录，但你也可以在任意的子目录中添加一个.gitignore文件，以覆盖更普遍的文件忽略规则。

- **.gitignore写法**

  ​	.gitignore文件应该按照规则编写。一个规则占一行，每个规则表示一个文件或目录需要被忽略。常用的规则格式如下：

  - 以`#`开头的行为注释，会被Git忽略。
  - 可以使用`*`符号匹配任意字符，比如`*.txt`可以匹配一类以".txt"结尾的文件。
  - 如果路径以`/`结尾，表示只忽略目录，不忽略同名的文件。比如`/foo/`表示忽略根目录下的`foo/`目录，但不包括`foo.txt`。
  - 如果路径以`/`开头，表示从项目根目录下开始匹配，比如`/*.txt`表示忽略根目录下的所有`.txt`文件。
  - 如果路径中包含`/`，表示忽略嵌套的目录，比如`foo/bar/`表示忽略`foo`目录下的`bar`目录。
  - 可以在行末添加`/`表示路径表示一个目录，而不是文件。比如`foo/`表示忽略所有同名目录，但不包括同名的文件。
  - 可以在行末添加`!`表示反选，即不被忽略，比如`!foo/bar/`表示不忽略`foo`目录下的`bar`目录。

  一个.gitignore文件可以包含多个规则，例如：

  ```
  # 忽略所有Build产生的文件
  build/
  
  # 不忽略libs下的foo目录
  !libs/foo
  
  # 忽略所有log文件
  *.log
   
  # 不忽略根目录下的log.txt文件
  !/log.txt
  
  # 忽略所有生成的jar包
  *.jar
  ```

- **POM**

1. `<dependencyManagement>` 

   <dependencyManagement>是 Maven POM（Project Object Model）的元素之一，它允许用户集中管理项目中使用的所有依赖项的版本号。`<dependencyManagement>` 元素只定义需要使用的依赖项和相应的版本，它并不实际引入这些依赖项。

   当 Maven 处理依赖项时，它会首先查找 `<dependencyManagement>`，并使用它来管理项目中所有依赖项的版本。这使得用户可以管理和维护项目中的所有依赖项版本，而不必在每个依赖项中指定版本号，提高了项目的可维护性。

2. `<profiles>` 

   <profiles>是 Maven POM（Project Object Model）的元素之一，它允许用户为不同的构建环境定义特定的构建配置。一个项目可能需要在不同的构建环境中使用不同的依赖项、插件或构建配置。Maven 允许用户使用 `<profiles>` 元素来定义不同的构建配置。

   在 `<profiles>` 元素中，用户可以定义一组 `<activation>` 规则，用于检测在该构建环境下该 `<profile>` 应该启用的条件。当这些条件被满足时，该 `<profile>` 中定义的所有配置都将被应用。

3. `<build>` 

   <build>是 Maven POM（Project Object Model）的元素之一，它是 Maven 构建的核心元素之一，用于定义项目的构建配置。`<build>` 元素包含了一系列子元素，如 `<plugins>`、`<pluginManagement>`、`<resources>`、`<testResources>`、`<extensions>` 等，这些子元素定义了项目的依赖、插件、构建输出目录、Maven 执行插件的配置等。

   通常，在 POM 文件中定义的构建配置只应该包含与构建相关的元素。以下是一些经常用到的子元素：

   - `<plugins>`：声明了 Maven 插件的配置和使用，这是使用 Maven 构建应用程序所必须的。如下示例：

     ```
     <plugins>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-compiler-plugin</artifactId>
         <version>3.8.0</version>
         <configuration>
           <source>1.8</source>
           <target>1.8</target>
         </configuration>
       </plugin>

       <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
         <version>2.4.4</version>
       </plugin>
     </plugins>
     ```

   - `<pluginManagement>`：这个元素用来管理 `<plugins>` 元素中配置的 Maven 插件。如下示例：

     ```
     <build>
       <pluginManagement>
         <plugins>
           <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-compiler-plugin</artifactId>
             <version>3.8.0</version>
             <configuration>
               <source>1.8</source>
               <target>1.8</target>
             </configuration>
           </plugin>

           <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
             <version>2.4.4</version>
           </plugin>
         </plugins>
       </pluginManagement>
     </build>
     ```

   - `<resources>`：定义了项目的资源目录，该目录下的所有资源将被编译到输出目录中。如下示例：

     ```
     <resources>
       <resource>
         <directory>src/main/resources</directory>
         <filtering>true</filtering>
         <includes>
           <include>**/*.xml</include>
           <include>**/*.json</include>
         </includes>
       </resource>
     </resources>
     ```

   - `<testResources>`：与 `<resources>` 元素类似，但是它会编译到测试输出目录中。如下示例：

     ```
     <testResources>
       <testResource>
         <directory>src/test/resources</directory>
         <filtering>true</filtering>
         <includes>
           <include>**/*.xml</include>
         </includes>
       </testResource>
     </testResources>
     ```

   - `<extensions>`：定义了 Maven 扩展的配置。如下示例：

     ```
     <extensions>
       <extension>
         <groupId>com.oracle.database.jdbc</groupId>
         <artifactId>ojdbc8</artifactId>
         <version>12.2.0.1</version>
       </extension>
     </extensions>
     ```

   在 Maven POM 中，`<build>` 元素是一个可选项。如果没有明确声明一个 `<build>` 元素，Maven 会使用默认值进行构建。此时，Maven 会使用默认的构建输出目录（`target` 目录），默认的编译目标版本（JDK 1.5），默认的源文件目录等。

- **stream流**

  在使用Stream之前，首先需要获取一个Stream对象，Stream对象可以通过将一个Collection接口中的数据流式化由顺序转为流来获取。

  简单获取方式：

  1. 通过集合对象获取：

  ```java
  List<String> list = Arrays.asList("a", "b", "c");
  Stream<String> stream1 = list.stream();
  ```

  2. 通过Arrays 中的静态方法获取数组对象来获取：

  ```java
  String[] strArray = new String[]{"a", "b", "c"};
  Stream<String> stream2 = Arrays.stream(strArray);
  ```

  3. 通过Stream中的静态方法 of 方法获取：

  ```java
  Stream<String> stream3 = Stream.of("a", "b", "c");
  ```

  获取到Stream对象后就可以开始对流数据实现简单的操作：

  1. forEach：遍历流中的元素

  ```java
  stream.forEach(System.out::println); //输出流中的所有元素
  ```

  2. filter：按照条件过滤

  ```java
  stream.filter(ele -> ele.startsWith("a")).forEach(System.out::println); //输出流中以"a"开始的所有元素
  ```

  3. map：将流中的每一个元素按照指定方式进行转换

  ```java
  stream.map(ele -> ele.toUpperCase()).forEach(System.out::println); //将流中的每个字母都转为大写字母输出
  ```

  4. limit：只获取流中的前n个元素

  ```java
  stream.limit(2).forEach(System.out::println); //只输出流中的前两个元素
  ```

  5. sorted：对流中的元素进行排序

  ```java
  stream.sorted().forEach(System.out::println); //对流中的元素进行自然排序
  ```

  6. distinct：去重

  ```java
  stream.distinct().forEach(System.out::println); //去除流中的重复元素
  ```

  除了以上这些常用的方法之外，Stream还提供了很多其他的方法。需要根据具体的业务需求来选择正确的方法进行操作。

  注意：每次调用Terminal Operations（输出类操作）都会走遍历整个流的流程。

  1. filter：过滤
  2. map：映射
  3. limit：限制返回个数
  4. skip：跳过指定数量的元素
  5. sorted：排序
  6. distinct：去重
  7. foreach：遍历
  8. count：计数
  9. collect：归约（将Stream中的数据转换为一个集合或者计算结果）

  总之，使用Stream可以通过简单的操作来处理集合中的数据，并且Stream API提供了大量方法支持多种操作，需要根据具体的业务需求来选择正确的方法进行操作。

- **CI/CD**

  ​	CI/CD是指持续集成/持续交付（Continuous Integration/Continuous Delivery）。

  持续集成是指软件开发团队通过对代码变更的频繁性进行自动化测试，以尽早发现和解决问题，从而缩短开发周期并提高软件质量。

  ​	持续交付则是指将经过持续集成和自动化测试后的软件交付给应用程序部署团队进行部署和验证。通过持续交付，软件开发团队可以更快地将新功能和改进的代码交付给最终用户，从而提高软件交付的速度和质量。

  CI/CD的关键是自动化。自动化可以帮助软件开发团队快速检测到问题并加快修复速度，提高软件质量，同时可以实现快速而稳定的软件部署，减少重复工作和减少出错率。

  ​	现在有很多CI/CD工具可以帮助开发团队实现自动化流程，如Jenkins、Travis CI、CircleCI等等。这些工具可以轻松集成版本控制、构建工具、自动化测试、部署工具等流程，从而全方位的提升软件开发和部署的效率，并提高应用程序的可靠性和可用性。

- **DMA控制器**

  ​	DMA控制器（Direct Memory Access Controller）是一种计算机系统中的专门硬件，其主要任务是在不占用CPU的情况下，直接将内存中的数据移动到外设（如硬盘、网卡、声卡等）中，从而提高传输速度和CPU利用率。

  ​	DMA控制器的工作原理是，在程序指定的内存区间和外设之间建立一个数据传输的通道，然后根据程序的要求，将数据直接从内存复制到外设的寄存器中，或者从外设的寄存器中读取数据并直接写入到内存中。在整个过程中，CPU不需要参与数据传输，而是集中精力处理其他任务，从而减轻CPU的负担，提高系统的整体性能。

- java GC

  https://sematext.com/blog/java-garbage-collection/

- WAR包与JAR包

  JAR包和WAR包都是Java中的打包格式，用于将Java程序打包成一个可执行的文件，但它们的应用场景和打包方式有所不同。

  JAR包是Java Archive的缩写，是一种用于打包Java类文件、资源文件和元数据的文件格式。JAR包通常用于打包Java库或Java应用程序，可以包含多个Java类文件和资源文件，可以被其他Java程序引用和调用。JAR包可以通过Java虚拟机（JVM）直接运行，也可以被打包成可执行的JAR文件，通过命令行或双击运行。

  WAR包是Web Application Archive的缩写，是一种用于打包Web应用程序的文件格式。WAR包通常包含了Web应用程序的所有文件，包括Java类文件、JSP文件、HTML文件、CSS文件、JavaScript文件、配置文件等。WAR包可以被部署到Web服务器上，通过Web服务器提供的Servlet容器（如Tomcat、Jetty等）来运行Web应用程序。

  JAR包和WAR包的主要区别在于应用场景和打包方式。JAR包主要用于打包Java库或Java应用程序，可以被其他Java程序引用和调用；而WAR包主要用于打包Web应用程序，可以被部署到Web服务器上运行。此外，JAR包和WAR包的打包方式也有所不同，JAR包通常使用Maven或Ant等构建工具进行打包，而WAR包通常使用Web应用程序打包工具（如Eclipse、IntelliJ IDEA等）进行打包。

- 

  





















































