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

