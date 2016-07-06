SLF4J, Simple Logging Facade for Java, serves as a simple facade or abstraction for various logging frameworks, such as java.util.logging, **logback** and log4j. SLF4J allows the end-user to plug in hte desired logging framework at *deployment* time.

### New Features ###

- Since 1.6.0 if no binding is found on the class path, then SLF4J will default to no-operation implementation.
- Since 1.7.0 Printing methods in the **Logger** interface now offer variants accept varargs instead of Object[]. This change implies that SLF4J requires JDK 1.5 or later.
- Since 1.7.5 Significant improvement in logger retrieval times. Given the extent of the improment, users are highly encouraged to migrate to SLF4J 1.7.5 or later.
- Since 1.7.9 By setting the `slf4j.detectLoggerNameMismatch` system property to true, SLF4J can automatically spot incorrectly named loggers.

SLF4J does not rely on any special class loader machinery. In fact, each SLF4J binding is hardwired at compile time to use one and only one specific logging framework. For example, the slf4j-log4j12-1.7.21.jar binding is bound at compile time to use log4j. In your code, in addition to slf4j-api-1.7.21.jar, you simply drop one and only one binding of your choice onto the appropriate class path location. Do not place more than one binding on your class path. Here is a graphical illustration of the general idea. 

<img src="http://www.slf4j.org/images/concrete-bindings.png">


<br>
### Executive summary ###
<table border="1" cellspacing="0" cellspan="5">
	<tr>
		<td>Advantage</td>
		<td>Description</td>
	</tr>
	<tr>
		<td>Select your logging framework at deployment time</td>
		<td>The desired logging framework can be plugged in at deployment time by inserting the appropriate jar file (binding) on your class path</td>
	</tr>
	<tr>
		<td>Fail-fast operation</td>
		<td>Due to the way that classes are loaded by the JVM, the framework binding will be verified automatically very early on. If SLF4J cannot find a binding on the class path it will emit a single warning message and default to no-operation implementation</td>
	</tr>
	<tr>
		<td>Bindings for popular logging frameworks</td>
		<td>SLF4J supports popular logging frameworks, namely log4j, java.util.logging, Simple logging and NOP. The logback project supports SLF4J natively.</td>
	</tr>
	<tr>
		<td>Briding legacy logging APIs</td>
		<td>The implementation of JCL over SLF4J, i.e jcl-over-slf4j.jar, will allow your project to migrate to SLF4J piecemeal, without breaking compatibility with existing software using JCL. Similarly, log4j-over-slf4j.jar and jul-to-slf4j modules will allow you to redirect log4j and respectively java.util.logging calls to SLF4J. See the page on Bridging legacy APIs(http://www.slf4j.org/legacy.html) for more details.</td>
	</tr>
	<tr>
		<td>Migrate your source code</td>
		<td>The slf4j-migrator utility can help you migrate your source to use SLF4J.</td>
	</tr>
	<tr>
		<td>Support for parameterized log messages</td>
		<td>All SLF4J bindings support parameterized log messages with significantly improved performance results.</td>
	</tr>

</table>


# Logback Project #
##Introduction to  Logback ##

Logbakc is intended as a successor to the popular log4j project. Logback is divided into three modules:

- logback-core
- logback-classic
- logback-access

Logback-core is the groundwork for the other two modules.

The logback-classic module can be assimilated to a significantly improved version of log4j. Moreover, logback-classic natively implements the SLF4J API so that you can readily switch back and forth between logback and other logging frameworks such as log4j or java.util.logging(JUL).

The logback-access module integrates with Servlet containers, such as Tomcat and Jetty, to provide HTTP-access log functionality. Note that you could easily build your own module on top of logback-core.

**Requirement**

Logback-classic module requires the presence of `slf4j-api.jar` and `logback-core.java` in addition to `logback-classic.jar`


**Configuration File**

logback will find the configuration file ini the classpath ordered as below:

- logback.groovy
- logback-test.xml
- logback.xml
- if found none of three files setting up default configuration.

By invoking the `StatusPrinter.print()` method can print its internal state

## Logback's architecture ##

### Logger, Appenders and Layouts ###
Logback is built upon three main classess: *Logger, Appender* and *Layout*

	Named Hierarchy
	
	A logger is said to be an ancestor for another logger if its 
	name followed by a dot is a prefix of the descendant logger name.
	A logger is said to be a parent of a child logger is there are no
	ancestors between itself and the descendant logger.



The `root` logger resides at the top of the logger hierarchy. It is expectional in that it is part of every hierarchy at its inception. Like every logger, it can be retrieved by its name, as follows:

	Logger rootLogger = LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);

The `Logger` interface is as below, it's printing methods associate with the logger level, like **TRACE -> DEBUG -> INFO -> WARN -> ERROR**

	package org.slf4j;
	
	public interface Logger {
	// omitted	
	// printing methods;
	public void trace(String message);
	public void debug(String message);
	public void info(String message);
	public void warn(String message);
	public void error(String message);
	
	}

The effective level for a given logger L, is equal to the first non-null level in its hierarchy, starting at L itself and proceeding upwards in the hierarchy towards the root logger.

To ensure that all loggers can eventually inherit a level, the root logger always has an assigned level. By default, this level is DEBUG.

**Calling the LoggerFactory.getLogger method with the same name will always return a reference to the exact same logger object**


### Appender Additivity ###
The output of a log statement of logger L will go to all the appenders in L and its ancestors. This is the meaning of the term **appender additivity**.

However, if an ancestor of logger L, say P, has the **additivity flag set to false**, then L's output will be directed to all the appenders in L and its ancestors up t and including P but not the appenders in any of the ancestors of P.

Loggers have their addivity flag set to true by default. 

### Parameterizd logging ###

using placeholder will be more efficient to print the message.

Only after evaluating whether to log or not, and only if the decision is positive, will the logger implementation format the message and replace the '{}' pair with the string value of entry.

	eg:
	Object entry = new SomeObject();
	logger.debug("The entry is {}.", entry);


### A peek under the hood ###

- Get the filter chain decision
- Apply the basic selection rule
- Create a LoggingEvent object
- Invoking appenders
- Formatting the output
- Sending out the LoggingEvent

**Under the Hood Sequence Diagram**

shows how the logging be done.
<img src="http://logback.qos.ch/manual/images/chapters/architecture/underTheHoodSequence2.gif">


### Performance ###

1. Logging performance when logging is turned off entirly
 
	You can turn off logging entirely by setting the level of the root logger to **Level.OFF**, the highest possible level.
2. The performance of deciding whether to log or not to log when logging is turned on.

	before accepting or denying a request based on the effective level, the logger can make a quasi-instantaneous decision, without needing to consult its ancestors.

3. Actual logging (formatting and writing to the output device)

	 The typical cost of actually logging is about 9 to 12 microseconds when logging to a file on the local machine. It goes up to several milliseconds when logging to a database on a remote server.


## Logback configuration ##

LogBack initialization steps:

- 1 Logback tries to find a file called *logback.groovy* in the classpatch
- 2 if no such file is found, logback tries to find a file called *logback-test.xml* in the classpath
- 3 if no such file is found, it checks for the file *logback.xml* in the classpath
- 4 if no such file is found, service-provider loading facility(introduced in JDK 1.6) is used to resolve the implementation of `com.qos.logback.classic.spi.Configurator` interface by looking up the file META-INF\services\ch.qos.logback.classic.spi.Configurator in the classpath. Its contents should specify the fully qualified class name of the desired Configurator implementation.
- 5 if none of the above succeeds, logback configures itself automatically using the `BasicConfiguraot` which will cause logging output to be directed to the console.

**FAST START-UP**: It takes about 100 miliseconds for Joran to parse a given logback configuration file. To shave off those miliseconds at aplication start up, you can use the service-provider loading facility (item 4 above) to load your own custom Configurator class with BasicConfigrator serving as a good starting point.


Except code that configures logback (if such code exists) client code does not need to depend on logback. Applications that use logback as their logging framework will have a compile-time dependency on SLF4J but not logback.

**A example of logback.xml**

<hr>
	<configuration>
		<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
	</configuration>
<hr>

If warning or errors occur during the parsing of the configuration file, logback will automatically print its internal status messages on the console.

You can print logback's internal status information as below:

	public static void main(String [] args){
		//assume SLF4J is bound to logback in the current environment
		LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
		// print logback's internal status
		StatusPrinter.print(lc);
		...
	}


## Appenders ##

### What is an Appender ###
Logback delegates the task of writing a logging event to components called appenders. Appenders must implement the `ch.qos.logback.core.Appender` interface. The salient methods of this interface are summarized below:

	package ch.qos.logback.core;
	
	import ch.qos.logback.core.spi.ContextAware;
	import ch.qos.logback.core.spi.FilterAttachable;
	import ch.qos.logback.core.spi.LifeCycle;

	public interface Appender<E> extends LifeCycle, ContextAware, FilterAttachable {
		public String getName();
		public void setName(String name);
		void doAppend(E event);
	}

**doAppend(E event)** is taking an object instance of type E as its only parameter. The actual type of E will vary depending on the logback module, Within the logback-classic module E would be of type `ILogingEvent`, and within the logback-access module it would be of type `AccessEvent`.


### AppenderBase ###

The `ch.qos.logback.core.AppenderBase` class is an abstract class implementing the Appender interface. It provides basic funcitonality shared by all appenders.

This AppenderBase do the logging on a synchronization way, if need unsynchronization way, `ch.qos.logback.core.UnsynchronizedAppenderBase ` would be your choice.

### Several appenders logback provided ###

- OutputStreamAppender
	- appends events to a java.io.OutputStream. Set *encoder* property can choice the manner in which the event is written to the underlying OutputStreamAppender

- ConsoleAppender
	- appends on the console
	- properties: *encoder, target, withJansi*
- FileAppender
	- a subclass of OutputStreamAppender, appends log events into a file.
	- properties: *append, encoder, file, prudent*
	- set the immediateFlush property of the underlying Encoder to false, close immediate flush
	- file is the name of the file to write to, can be uniquely named.
- RollingFileAppender
	- RollingFileAppender extends FileAppender with the capability to rollover log files.
	- set **rollingPolicy** to change the behavior when rollover occurs
	- set **triggeringPolicy** tell RollingFileAppender when to activate the rollover procedure.
	
- SocketAppender and SSLSocketAppender
	- The appenders covered thus far are only able to log to local resources. In contrast, the SocketAppender is designed to log to a remote entity by transmitting serialized ILoggingEvent instances over the wire. When using SocketAppender logging events on the wire are sent in the clear. However, when using SSLSocketAppender, logging events are delivered over a secure channel.
	
- SMTPAppender
	- SMTPAppender use the JavaMail API to send emails, it does not have an timeout property.
	- if need set timeout, rewrite the SMTPAppender, add **mail.smtp.connectiontimeout** and **mail.smtp.timeout** property to session in the start() method. The propertie will be used by JAVAMail API to set the connection timeout and send timeout.
	- set **Marker** to send on marker.
	- set **Evalutor** to send on evaluator
	- support MDCDiscriminator

- DBAppender
	- The DBAppender inserts logging events into three database tables in a format independent of the Java programming language.

- SyslogAppender
	- The syslog protocol is a very simple protocol: a syslog sender sends a small message to a syslog receiver. The receiver is commonly called syslog daemon or syslog server. Logback can send messages to a remote syslog daemon

- SiftingAppender
	- As its name implies, a SiftingAppender can be used to separate (or sift) logging according to a given runtime attribute. For example, SiftingAppender can separate logging events according to user sessions, so that the logs generated by different users go into distinct log files, one log file per user.

- AsyncAppender
	- AsyncAppender logs ILoggingEvents asynchronously. It acts solely as an event dispatcher and must therefore reference another appender in order to do anything useful.
	


**note that the appenders above located at Logback-core, logback-classic, logback-access separately**

## Encoders ##

Encoders are responsible for transforming an event into a byte array as well as writing out that byte array into an OutputStream.

Encoders are responsible for transforming an incoming event into a byte array and writing out the resulting byte array onto the appropriate OutputStream. Thus, encoders have total control of what and when bytes gets written to the OutputStream maintained by the owning appender. Here is the Encoder interface:

## Layouts ##
Layouts are logback components responsible for transforming an incoming event into a String. 

	public interface Layout<E> extends ContextAware, LifeCycle {
		String doLayout(E event);
		String getFileHeader();
		String getPresentationHeader();
		String getFileFooter();
		String getPresentationFooter();
		String getContentType();
	}


Configuring custom layout

	<configuration>
		<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
	    	<encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
	      		<layout class="chapters.layouts.MySampleLayout" />
	    	</encoder>
		</appender>
		<root level="DEBUG">
	    	<appender-ref ref="STDOUT" />
		</root>
	</configuration>


## Filters ##
Logback filters are based on ternary logic allowing them to be assembled or chained together to compose an arbitrarily complex filtering policy. They are largely inspired by Linux's iptables.

Creating your own filter is easy. All you have to do is extend the Filter abstract class and implement the decide() method.

	package chapters.filters;

import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class SampleFilter extends Filter<ILoggingEvent> {

	@Override
	public FilterReply decide(ILoggingEvent event) {    
	    if (event.getMessage().contains("sample")) {
	      return FilterReply.ACCEPT;
	    } else {
	      return FilterReply.NEUTRAL;
	    }
 	 }
	}


Several Filters

- LevelFilter

		<filter class="ch.qos.logback.classic.filter.LevelFilter">
      		<level>INFO</level>
      		<onMatch>ACCEPT</onMatch>
      		<onMismatch>DENY</onMismatch>
   		 </filter>

- ThresholdFilter
	
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      		<level>INFO</level>
   		 </filter>

- EvaluatorFilter

	EvaluatorFilter is a generic filter encapsulating an EventEvaluator. As the name suggests, an EventEvaluator evaluates whether a given criteria is met for a given event. On match and on mismatch, the hosting EvaluatorFilter will return the value specified by the onMatch or onMismatch properties respectively.

- TurboFilters

	TurboFilter objects all extend the TurboFilter abstract class. Like the regular filters, they use ternary logic to return their evaluation of the logging event.

	**More importantly, they are called before the LoggingEvent object creation. TurboFilter objects do not require the instantiation of a logging event to filter a logging request. As such, turbo filters are intended for high performance filtering of logging events, even before the events are created.**


**for more detials, refer to [http://logback.qos.ch/manual/](http://logback.qos.ch/manual/ "The logback manual")**
