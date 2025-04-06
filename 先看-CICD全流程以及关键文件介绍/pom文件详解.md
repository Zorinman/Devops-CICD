
`pom.xml` 是 Maven 项目的核心配置文件，POM 代表 **Project Object Model（项目对象模型）**。它定义了项目的基本信息、依赖关系、构建配置等内容，是 Maven 构建和管理项目的基础。

以下是 `pom.xml` 的主要作用：

1. **项目基本信息**  
   `pom.xml` 包含项目的基本信息，如 `groupId`（组织标识符）、`artifactId`（项目标识符）、`version`（版本号）等。这些信息唯一标识一个 Maven 项目。

2.**指定依赖**
为maven指定了要下载哪些依赖和插件

总结来说，`pom.xml` 是 Maven 项目管理的核心，通过它可以实现依赖管理、构建配置、模块化开发等功能，大大简化了 Java 项目的开发和维护流程。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 定义 Maven 项目的模型版本 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 指定父项目，继承 Spring Boot 的 starter parent -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.9</version>
        <relativePath/> <!-- 从仓库中查找父项目 -->
    </parent>

    <!-- 定义项目的唯一标识符 -->
    <groupId>cn.wolfcode</groupId> <!-- 项目组 ID -->
    <artifactId>k8s-cicd-demo</artifactId> <!-- 项目 Artifact ID -->
    <version>1.0.0</version> <!-- 项目版本 -->
    <name>k8s-cicd-demo</name> <!-- 项目名称 -->

    <!-- 项目描述信息 -->
    <description>K8S CICD 演示项目</description>

    <!-- 项目属性定义 -->
    <properties>
        <java.version>1.8</java.version> <!-- 指定 Java 版本 -->
    </properties>

    <!-- 项目依赖定义 -->
    <dependencies>
        <!-- Spring Boot Web Starter，用于构建 Web 应用 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Lombok，用于简化 Java 代码 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional> <!-- 可选依赖 -->
        </dependency>

        <!-- Spring Boot Test Starter，用于测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope> <!-- 测试范围依赖 -->
        </dependency>
    </dependencies>

    <!-- 构建配置 -->
    <build>
        <!-- 插件管理 -->
        <pluginManagement>
            <plugins>
                <!-- SonarQube 插件，用于代码质量分析 -->
                <plugin>
                    <groupId>org.sonarsource.scanner.maven</groupId>
                    <artifactId>sonar-maven-plugin</artifactId>
                    <version>3.9.1.2184</version>
                </plugin>
            </plugins>
        </pluginManagement>

        <!-- 插件配置 -->
        <plugins>
            <!-- Spring Boot Maven 插件，用于打包和运行 Spring Boot 应用 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- 排除特定依赖 -->
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```
