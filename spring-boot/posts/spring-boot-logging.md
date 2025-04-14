# Spring Boot Logging Configuration Guide

*Published on February 15, 2024*

## Overview

This guide provides a comprehensive overview of logging configuration in Spring Boot applications. Logging is a critical aspect of application development and maintenance, helping developers track application behavior, debug issues, and monitor system health. Spring Boot provides excellent support for various logging frameworks like Logback, Log4j2, and SLF4J, making it easy to implement robust logging solutions.

## Table of Contents

1. [What is Logging?](#what-is-logging)
2. [Spring Boot Logging Support](#spring-boot-logging-support)
3. [Getting Started](#getting-started)
4. [Dependencies](#dependencies)
5. [Configuration](#configuration)
6. [Implementation](#implementation)
7. [Logging Levels](#logging-levels)
8. [Advanced Configuration](#advanced-configuration)
9. [Best Practices](#best-practices)
10. [References](#references)

## What is Logging?

Logging is the process of recording events, messages, and data about an application's execution. It serves several important purposes:

- Debugging and troubleshooting
- Monitoring application health
- Tracking user activities
- Auditing and compliance
- Performance analysis
- Security monitoring

## Spring Boot Logging Support

Spring Boot provides comprehensive logging support through:

- Default Logback implementation
- Log4j2 support
- SLF4J facade
- Easy configuration through properties
- Custom configuration files
- Multiple appenders and layouts
- Log rotation and archiving

## Getting Started

### Dependencies

Spring Boot includes logging dependencies by default. For Logback (default):

```xml
<!-- Included by default in spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

For Log4j2:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

## Configuration

### Basic Configuration in application.properties

```properties
# Logging level
logging.level.root=INFO
logging.level.com.yourpackage=DEBUG

# Log file configuration
logging.file.name=application.log
logging.file.path=/var/log/

# Log pattern
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
```

## Implementation

### Using SLF4J in Your Code

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class MyController {
    private static final Logger logger = LoggerFactory.getLogger(MyController.class);
    
    @GetMapping("/example")
    public String example() {
        logger.trace("This is a TRACE message");
        logger.debug("This is a DEBUG message");
        logger.info("This is an INFO message");
        logger.warn("This is a WARN message");
        logger.error("This is an ERROR message");
        
        return "Check the logs!";
    }
}
```

## Logging Levels

The logging levels in order of severity are:
- TRACE: Very detailed information for debugging
- DEBUG: Detailed information for debugging
- INFO: Important business process information
- WARN: Potentially harmful situations
- ERROR: Errors that need immediate attention

## Advanced Configuration

### Custom Logback Configuration

Create a `logback-spring.xml` file in your `src/main/resources` directory:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_PATH" value="logs"/>
    <property name="LOG_FILE" value="application"/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${LOG_FILE}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/${LOG_FILE}.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### Log4j2 Configuration

Create a `log4j2.xml` file in your `src/main/resources` directory:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Properties>
        <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} %5p [%t] %c{1}:%L - %m%n</Property>
        <Property name="LOG_PATH">logs</Property>
        <Property name="LOG_FILE">application</Property>
    </Properties>
    
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}"/>
        </Console>
        
        <RollingFile name="File" fileName="${LOG_PATH}/${LOG_FILE}.log"
                     filePattern="${LOG_PATH}/${LOG_FILE}-%d{yyyy-MM-dd}.log">
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>
    </Appenders>
    
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```

## Best Practices

1. **Use appropriate logging levels**:
   - ERROR: For errors that need immediate attention
   - WARN: For potentially harmful situations
   - INFO: For important business process information
   - DEBUG: For detailed information useful for debugging
   - TRACE: For very detailed information

2. **Use parameterized logging**:
   ```java
   // Good
   logger.debug("User {} logged in from {}", username, ipAddress);
   
   // Bad
   logger.debug("User " + username + " logged in from " + ipAddress);
   ```

3. **Configure different log levels for different packages**:
   ```properties
   logging.level.com.yourpackage=DEBUG
   logging.level.org.springframework=INFO
   logging.level.org.hibernate=WARN
   ```

4. **Use MDC (Mapped Diagnostic Context) for contextual information**:
   ```java
   MDC.put("userId", userId);
   logger.info("User action performed");
   MDC.clear();
   ```

5. **Implement log rotation and archiving**:
   - Configure appropriate file sizes and retention periods
   - Use time-based rotation for better organization
   - Consider compression for archived logs

6. **Include relevant context in log messages**:
   - Timestamps
   - Thread information
   - Log level
   - Class and method names
   - Request IDs for distributed tracing

7. **Monitor log levels in production**:
   - Keep ERROR and WARN levels active
   - Adjust INFO level based on needs
   - Disable DEBUG and TRACE in production

## References

- [Spring Boot Logging Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)
- [Logback Documentation](http://logback.qos.ch/manual/index.html)
- [Log4j2 Documentation](https://logging.apache.org/log4j/2.x/manual/index.html)
- [SLF4J Documentation](http://www.slf4j.org/manual.html)
- [Spring Boot Logging Guide](https://www.baeldung.com/spring-boot-logging)
