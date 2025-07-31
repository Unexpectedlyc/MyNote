# MCP搭建

## MCP三种通信方式

三大通信模式差异

STDIO模式：适合快速开发和本地调试，但不适合发布生产环境

SSE模式：适合需要实时推送的远程服务

Streamable HTTP模式：MCP的未来方向，尤其在云原生和无状态架构中具有显著优势

|特性|STDIO|SSE|Streamable HTTP|
|:----|:----|:----|:----|
|通信方向|单向（请求-响应）|单向（服务端→客户端）|双向流|
|协议基础|操作系统内建功能|基于HTTP协议|基于HTTP协议|
|连接方式|本地进程之间，通常为短连接|持久HTTP连接，自动重连|持久HTTP连接|
|数据形式|纯文本|富媒体（图/JSON/视频）|任意二进制/文本流|
|延迟要求|低（同步阻塞）|中（实时推送）|低（流式传输）|
|适用场景|命令行工具、脚本交互|实时数据推送|音视频流、大文件传输|
|浏览器支持|不适用（主要为本地程序）|支持大部分现代浏览器|支持所有现代浏览器|
|并发能力|低（单线程）|中（依赖服务器配置）|高（无状态化）|
|安全性|高（本地隔离）|中（依赖HTTPS）|高（支持HTTPS、会话隔离）|

## Spring AI的maven依赖配置

基于jdk17+spring boot3.3+spring ai-1.0.0

maven版本3.9.9

由于 Spring AI 尚未发布到 Maven 中央仓库，需要手动添加快照仓库和依赖项

下面是pom.xml

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.mapgis.zondy</groupId>
    <artifactId>MapgisWebMcpServerDemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>MapgisWebMcpServerDemo</name>
    <description>MapgisWebMcpServerDemo</description>
    <properties>
        <java.version>17</java.version>
        <spring-ai.version>1.0.0</spring-ai.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-mcp-server-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-mcp-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-mcp-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>${spring-ai.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <releases>
                <enabled>false</enabled>
            </releases>
        </repository>
        <repository>
            <id>central-portal-snapshots</id>
            <name>Central Portal Snapshots</name>
            <url>https://central.sonatype.com/repository/maven-snapshots/</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
        
```
Spring AI的application.properties基本配置
```plain
spring.application.name=MapgisWebMcpServerDemo
# Server identification
spring.ai.mcp.server.name=my-weather-server
spring.ai.mcp.server.version=0.0.1

# Server type (SYNC/ASYNC)
spring.ai.mcp.server.type=SYNC

# Transport configuration
spring.ai.mcp.server.stdio=false
spring.ai.mcp.server.sse-message-endpoint=/mcp/message

# Change notifications
spring.ai.mcp.server.resource-change-notification=true
spring.ai.mcp.server.tool-change-notification=true
spring.ai.mcp.server.prompt-change-notification=true

# Logging (required for STDIO transport)
spring.main.banner-mode=off
logging.file.name=./target/starter-webflux-server.log

```
更多详细配置请参考spring ai官方文档[MCP Server Boot Starter :: Spring AI Reference](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)

