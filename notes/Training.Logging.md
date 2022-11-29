---
id: oji664drtxpoum2bxu4gjo3
title: Logging
desc: ''
updated: 1669731610693
created: 1669629977276
---
Usually most use `System.out.println()` methods to output logging messages to the terminal.
There are many logging frameworks like *log4j* and *logback* that can add enhancements to logigng like differing levels of logging output.

Simple Logging Facade for Java (SLF4J) is an abstraction for various logging frameworks allowing you to plug in the desired logging framework at *deployment time*.

Logging Levels

Level   | Purpose
------- | ---------
Trace   | Lowest, most granular, could include info as state of variables in complex loop
Debug   | Useful info, to help inform about program flow or provide state information
Info    | General info about what is happening, default level
Warning | Information about problems or inconsistencies that don't prevent program from working but may be concerning
Error   | Fatal errors, usually that prevent program or process running further


```Java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class className {
    private static final Logger LOGGER = LoggerFactory.getLogger(CAPValuationCalculator.class);

    public returnType methodName(String var1, int var2) {
        LOGGER.debug("Entering methodName(var1={},var2={})", var1, var2);

        if (errorCondition) {
            LOGGER.error("Cannot have negative val on the var2 (var2={})", var1);
            throw new whateverException("The value must be positive");
        }
        // some logic

        LOGGER.info("Exiting methodName with var1={}", var1);
        return;
    }

}
```


### Configuration File
[[Config File at bottom of auto config | https://logging.apache.org/log4j/2.x/manual/configuration.html#AutomaticConfiguration]]

``` XML
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```
The root level determines what errors are output to console, changing to trace for example would output all logger calls. This will all output to the console.

There are some additions like:
- Add a package specific logger for the pricing package that has a lower logging level than the root logger and check that classes in that package now log at that level
- Use an appender that writes to a file rather than the console 
- Change the [[message format | https://logging.apache.org/log4j/2.x/manual/layouts.html]]

- Use asynchronous appender for performance improvements
    - this will require additional dependency for disruptor

``` XML
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="debug" name="Pricing Package test" packages="com.apakgroup.training.tutorial.pricing">
    <Properties>
        <Property name="fileName">target/test.log</Property>
    </Properties>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        <File name="File" fileName="${fileName}">
            <PatternLayout pattern="%d{HH:mm:ss} %c{1} %p - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="debug">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```