# Logging to File

## Learning Goals

- Log to files instead of the console.

## Introduction

So far all of our output has been directed to the console. This isn't ideal,
since the console is not persistent and requires someone to be physically
looking at the system while the console output happens to see it and monitor it.

## Log to File

A simple way to overcome this limitation is to ask our logging system to log
messages to file. This can be done in the same `application.properties` file
where we've made all our configuration changes so far:

```properties
logging.level.root=INFO
logging.level.com.flatiron.spring=TRACE
logging.file.name=flatiron-spring.log
```

You can now restart your application and see that it creates a file with the
name you specified in the root folder of your project. Make a request to your
`hello()` endpoint and you will see that the `TRACE` level statements are also
appended to your file.

### Appenders

SLF4J supports the concept of appenders. So far, we've used the basic versions
of the `ConsoleAdapter` and the `FileAppender` and we've configured them using a
very simplified version of their possible configuration in
`application.properties`.

We will now dive into the full configuration for the actual logging library that
is the default implementation of the SLF4J specific for the Spring Framework:
Logback.

First, let's comment our logging configuration from `application.properties`:

```properties
#logging.level.root=TRACE
#logging.level.com.flatiron.spring=TRACE
#logging.file.name=flatiron-spring.log
```

Now let's create the configuration file for Logback - it's an XML-based file
that is ready by the Spring Framework and used to initialize the Lockback
library, as long as it's named "logback.xml" and is in the classpath. So let's
create a file with the name and put it in our `src/main/resouces` folder. The
base version will look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
    <logger name="com.flatiron.spring" level="TRACE"/>
</configuration>
```

Let's break it down:

1. The `include` elements are to specify the base configurations we want to use
   as baselines. Here we specify the "defaults" and the "console appender"
   configurations, which are provided by default by the Spring Framework
2. We define the root logging level, and we tell Logback to use the "console"
   appender, which will output to our application console
3. We then define a custom logging level, as we did before

Running the application in this configuration will give you similar output to
before, but now we have the foundation we need in order to start defining
additional appenders.

We will define 2 appenders:

1. A "console" appender, which is the default appender we've been using, but
   defining explicitly here will allow us to customize it.
2. A "file" appender, which will allow us to direct the output of our
   application to a file

Here is the new configuration we will start with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <appender name="Console-Appender" class="ch.qos.logback.core.ConsoleAppender">
        <layout>
            <pattern>CONSOLE: %msg%n</pattern>
        </layout>
    </appender>
    <appender name="File-Appender" class="ch.qos.logback.core.FileAppender">
        <file>flatiron-spring.log</file>
        <encoder>
            <pattern>FILE: %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="Console-Appender" />
        <appender-ref ref="File-Appender" />
    </root>
    <logger name="com.flatiron.spring" level="TRACE">
        <appender-ref ref="Console-Appender" />
        <appender-ref ref="File-Appender" />
    </logger>
</configuration>
```

Let's break down this new configuration file:

1. We include the default configurations for logback itself and the console and
   file appenders - this will allow us to use environment variables that are
   defined in those configuration files. Note: you can look through your
   classpath for these files and look at the configuration to see what values
   they define.
2. We define a "Console-Appender" and give it a very simple pattern - more on
   patterns below
3. We define a "File-Appender" and also give it a similar simple pattern
4. We then have the same loggers defined as before, except that we now tell both
   to append to both appenders

Run the application with this simple configuration, and you should get output
similar to this in `flatiron-spring.log`:

```java
FILE: Starting FlatironSpringApplication using Java 17.0.3 on asterix.lan with PID 30735 (/Users/dbash/sandbox/flatiron/flatiron-spring/build/classes/java/main started by dbash in /Users/dbash/sandbox/flatiron/flatiron-spring)
FILE: Starting FlatironSpringApplication using Java 17.0.3 on asterix.lan with PID 30735 (/Users/dbash/sandbox/flatiron/flatiron-spring/build/classes/java/main started by dbash in /Users/dbash/sandbox/flatiron/flatiron-spring)
FILE: Running with Spring Boot v2.7.1, Spring v5.3.21
FILE: Running with Spring Boot v2.7.1, Spring v5.3.21
FILE: No active profile set, falling back to 1 default profile: "default"
FILE: No active profile set, falling back to 1 default profile: "default"
FILE: Tomcat initialized with port(s): 8080 (http)
```

And output similar to this on your console:

```java
CONSOLE: Starting FlatironSpringApplication using Java 17.0.3 on asterix.lan with PID 30735 (/Users/dbash/sandbox/flatiron/flatiron-spring/build/classes/java/main started by dbash in /Users/dbash/sandbox/flatiron/flatiron-spring)
CONSOLE: Starting FlatironSpringApplication using Java 17.0.3 on asterix.lan with PID 30735 (/Users/dbash/sandbox/flatiron/flatiron-spring/build/classes/java/main started by dbash in /Users/dbash/sandbox/flatiron/flatiron-spring)
CONSOLE: Running with Spring Boot v2.7.1, Spring v5.3.21
CONSOLE: Running with Spring Boot v2.7.1, Spring v5.3.21
CONSOLE: No active profile set, falling back to 1 default profile: "default"
CONSOLE: No active profile set, falling back to 1 default profile: "default"
CONSOLE: Tomcat initialized with port(s): 8080 (http)
```

As you can see, the format of the output is very similar between the file and
the console appender, because we specified very similar patterns. In the next
section, we will look at patterns in more detail.

#### Appender Pattern

Let's break down the pattern we used for the file appender:

`FILE: %msg%n`

A conversion pattern is made up of "free text" and "format control expressions":

1. Free text is anything you want to have appear verbatim in your output - in
   our case, we've specified that each line of output should start with the
   text: "FILE: ". Note that for the console output pattern, we've specified
   that each line of output should start with the text: "CONSOLE: "
2. Format control expressions are a way to give the logger instructions. Each
   instruction starts with the "%" character and is then following by a
   supported instruction. In our simple example, we used 2 instructions:
   1. "%msg": this is a reference to the message that was passed into the
      logger, so it outputs the message itself
   2. "%n": this tells the logger to start a new line
3. Since "%" has a specific meaning in patterns, it needs to be "escaped" in
   order to be used as a literal. This is done by using "\%" wherever you would
   like to have a literal "%" be included in the output of the logger
4. Some format control expressions can take parameters, in which case the
   parameters are including in "{}" directly after the expression

> Format control expressions usually have a "long" and a "short" version. For
> example, the format control expression for outputting a date is `%date{yyyy}`,
> but it can also be used as `%d{yyyy}`. For the benefit of readability, we will
> use the long form in this section.

There are many format control expressions available for pattern definitions in
your logger configuration. Here is an example of a more useful configuration and
an explanation of the format control expressions it uses:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <appender name="Console-Appender" class="ch.qos.logback.core.ConsoleAppender">
        <layout>
            <pattern>CONSOLE: %msg%n</pattern>
        </layout>
    </appender>
    <appender name="File-Appender" class="ch.qos.logback.core.FileAppender">
        <file>flatiron-spring.log</file>
        <encoder>
            <pattern>%date{yyyy-MM-dd HH:mm:ss.SSS}-%6level --- [%thread] %-40.40logger{39} : %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="Console-Appender" />
        <appender-ref ref="File-Appender" />
    </root>
    <logger name="com.flatiron.spring" level="TRACE">
        <appender-ref ref="Console-Appender" />
        <appender-ref ref="File-Appender" />
    </logger>
</configuration>
```

The new pattern for our file appender has several new instructions:

1. `%date`: this prints out the date in the specified format. Here, we request a
   year-month-day format for the date, and a hour:minute:seconds.milliseconds
   format for the time
2. `%level`: this prints out the logging level associated with the logging
   statement for this line. It is also used in conjunction with a "modifier". In
   this case, the modifier is "6", which tells the logger that the minimum width
   of this field in the output line should be 6 characters, and to use left
   padding if the actual level is length than 6 characters long. There is no
   maximum here, so if the level field is longer than 6 characters, it would
   still be displayed in its entirety.
3. The " --- [" characters are literals that will appear exactly like that in
   the output
4. `%thread`: displays the name of the thread that this logging statement came
   from
5. The "] " characters are literals
6. `%logger`: displays the name of the class that this logging statement came
   from. It is used with several modifiers:
   1. `-40` means right pad with spaces if the name of the logger is shorter
      than 40 characters
   2. `.40` means truncate from the beginning if the name of the logger is
      longer than 40 characters
   3. `{39}` is a parameter to the logger instruction to tell it to shorten the
      name of the class so that it fits within 39 characters. This is usually
      done by the logger instruction by shortening the name of the packages in
      the fully qualified name of the class.
7. The " : " characters are literals
8. `%msg`: the actual message passed into the logging statement
9. `%n`: requesting a new line in the log

Running your application with this configuration will give you output similar to
this:

```java
2022-07-13 01:23:13.459-  INFO --- [main] c.f.s.F.FlatironSpringApplication        : Starting FlatironSpringApplication using Java 17.0.3 on asterix.lan with PID 30913 (/Users/dbash/sandbox/flatiron/flatiron-spring/build/classes/java/main started by dbash in /Users/dbash/sandbox/flatiron/flatiron-spring)
2022-07-13 01:23:13.459-  INFO --- [main] c.f.s.F.FlatironSpringApplication        : Starting FlatironSpringApplication using Java 17.0.3 on asterix.lan with PID 30913 (/Users/dbash/sandbox/flatiron/flatiron-spring/build/classes/java/main started by dbash in /Users/dbash/sandbox/flatiron/flatiron-spring)
2022-07-13 01:23:13.460- DEBUG --- [main] c.f.s.F.FlatironSpringApplication        : Running with Spring Boot v2.7.1, Spring v5.3.21
2022-07-13 01:23:13.460- DEBUG --- [main] c.f.s.F.FlatironSpringApplication        : Running with Spring Boot v2.7.1, Spring v5.3.21
2022-07-13 01:23:13.460-  INFO --- [main] c.f.s.F.FlatironSpringApplication        : No active profile set, falling back to 1 default profile: "default"
2022-07-13 01:23:13.460-  INFO --- [main] c.f.s.F.FlatironSpringApplication        : No active profile set, falling back to 1 default profile: "default"
2022-07-13 01:23:13.824-  INFO --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
```
