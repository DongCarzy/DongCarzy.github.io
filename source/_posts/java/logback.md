---
layout: post
title: logback
date: 2018-12-06 12:03:32
tags: java
categories: java
---

# Logback

`springboot` 中通过 `logback-spring.xml`自定义配置 `logback` 日志

## 文件名

在 `springboot` 系统中，springboot 会默认加载 `resource` 下的 `logback.xml` 或者 `logback-spring.xml`， 但是建议将文件名命名为 `logback-spring.xml`， 因为`logback.xml` 会比 `application.properties` 更早加载， 就会导致在你的日志配置文档中不能使用 `application.properties` 中的变量

### 变量对应关系

| application.properties            | logback-spring.xml            |
| --------------------------------- | ----------------------------- |
| logging.exception-conversion-word | LOG_EXCEPTION_CONVERSION_WORD |
| logging.file                      | LOG_FILE                      |
| logging.file.max-size             | LOG_FILE_MAX_SIZE             |
| logging.file.max-history          | LOG_FILE_MAX_HISTORY          |
| logging.path                      | LOG_PATH                      |
| logging.pattern.console           | CONSOLE_LOG_PATTERN           |
| logging.pattern.dateformat        | LOG_DATEFORMAT_PATTERN        |
| logging.pattern.file              | FILE_LOG_PATTERN              |
| logging.pattern.level             | LOG_LEVEL_PATTERN             |
| PID                               | PID                           |

## 彩色输出

* 通过 `conversionRule` 引入 `springboot` 的规则, 将通过 `ansi` 码表示我们的颜色
* 在 `application.properties` 中开启 `ansi` 功能 `spring.output.ansi.enabled=always`

相关配置如下:

```xml
 <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <property name="FILE_LOG_PATTERN"
              value="${FILE_LOG_PATTERN:-%d{yyyy-MM-dd HH:mm:ss.SSS} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
```

## 日志回滚打包

通过 `rollingPolicy` 标签来达到回滚打包，规则有很多中，可参考 `logbak`官网，以下是我完整的配置

* `${LOG_PATH}` 是引用的 `application.properties`中的属性 `logging.path`
* 输出格式 `pattern` 可自行参考官网，这里我是按照 `springboot` 默认格式输出，来源： `org/springframework/boot/logging/logback/defaults.xml` 这个文件
* 最终效果， 当日志 `spring.log` 文件达到 `300M` 照是， 就会被压缩成 `zip` 包，文件名称为 `%d{yyyy-MM-dd}.%i.log.zip`, 年月日+当天压缩包的序号.log.zip

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true">
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <property name="FILE_LOG_PATTERN"
              value="${FILE_LOG_PATTERN:-%d{yyyy-MM-dd HH:mm:ss.SSS} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/spring.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_PATH}/%d{yyyy-MM-dd}.%i.log.zip</FileNamePattern>
            <!--日志文件保留个数-->
            <MaxHistory>50</MaxHistory>
            <!-- 日志总保存量为10GB -->
            <totalSizeCap>10GB</totalSizeCap>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!--活动文件大小 -->
                <maxFileSize>300MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- 日志输出默认级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILE"/>
    </root>

    <logger name="com.tonmx" level="DEBUG"/>

</configuration>
```