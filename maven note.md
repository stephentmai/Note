# Maven常用命令
|命令|描述|
|----|----|
|mvn -version|显示版本信息|
|mvn clean|清理项目生产的临时文件，一般是模块下的target目录|
|mvn compile|编译源代码，一般编译模块下的src/main/java目录|
|mvn package|项目打包工具，会在模块下的target目录生成jar或war等文件|
|mvn test|测试命令，或执行src/test/java下junit的测试用例|
|mvn install|将打包的jar/war文件复制到你的本地仓库中，供其他模块使用|
|mvn deploy|将打包的文件发布到远程参考，提供其他人员进行下载依赖|
|mvn site|生成项目相关信息的网站|
|mvn eclipse:eclipse|将项目转化为Eclipse项目|
|mvn dependency:tree|打印出项目的整个依赖树| 
|mvn archetype:generate|创建Maven的普通java项目|
|mvn tomcat7:run|在tomcat容器中运行web应用|
|mvn jetty:run|调用Jetty插件的Run目标在Jetty Servlet容器中启动web应用|

华为maven仓库
```xml
<mirror>

    <id>huaweicloud</id>

    <mirrorOf>central</mirrorOf>

    <url>https://repo.huaweicloud.com/repository/maven/</url>
    
</mirror>
```

# Maven问题
## “不支持源选项5。请使用7或更高版本。”

（1）暂时解决
在pom.xml文件中指定的jdk版本，运行另一个项目，需要在另一个项目的pom.xml文件中再配置一个，每换一个项目，都需要在pom.xml中配置一个，所以是暂时解决的。

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
 </properties>
 ```

（2）永久解决
在maven配置文件settings.xml文件中在每次运行maven用指定的jdk版本

把下面的代码放到标签的中间才能生效
 ```xml
<profile>  
     <id>jdk-1.8</id>
     <activation>
         <activeByDefault>true</activeByDefault>  
         <jdk>1.8</jdk>  
     </activation>
     <properties>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
         <maven.compiler.source>1.8</maven.compiler.source>
         <maven.compiler.target>1.8</maven.compiler.target>
     </properties>
</profile>
 ```

## “Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test”
```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.12.4</version>
        <configuration>
          <skipTests>true</skipTests>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ```
