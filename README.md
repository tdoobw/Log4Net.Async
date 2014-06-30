Log4Net.Async
=============

[![Build status](https://ci.appveyor.com/api/projects/status/fpn8apunhe0fr1y3)](https://ci.appveyor.com/project/cjbhaines/log4net-async)

This library provides asynchronous Log4Net logging which can massively improve application performance if you are logging into a slow database for example. The basic concept with the library is that is uses a ring buffer with a default limit of 1000 to store pending logging events and will process them on a dedicated background thread. If the head of the buffer overtakes the tail lossy logging occurs and the oldest entries will be lost. This should never be a problem unless you are logging vast amounts of log messages to slow appenders in which case you probably need to readdress your logging.

There are 2 methods to perform async logging using this library:

1) Use an Async forwarder which wraps the existing log4net appenders and forwards their output to the async buffer [Preferred]

2) Use the AsyncAdoAppender and AsyncRollingFileAppender instead of the log4net library versions


Async Forwarder
=============
###AsyncForwardingAppender

This is easily set up and wraps multiple appenders like so:

	<appender name="rollingFile" type="log4net.Appender.RollingFileAppender">
		...
	</appender>
	
	<appender name="adoNet" type="log4net.Appender.AdoNetAppender">
		...
	</appender>
	
	<appender name="asyncForwarder" type="Log4Net.Async.AsyncForwardingAppender,Log4Net.Async">
		<appender-ref ref="rollingFile" />
		<appender-ref ref="adoNet" />
	</appender>

	<root>
		<appender-ref ref="asyncForwarder" />
	</root>
  
Note: If you are using many appenders such as Console, Gelf, File, etc along with an AdoNetAppender and your database is not highly optimized I would recommend using a dedicated forwarder for the AdoNetAppender.
  
Async Appenders
=============
###AsyncAdoAppender

	<appender name="asyncAdoNet" type="Log4Net.Async.AsyncAdoAppender,Log4Net.Async">
		...
	</appender>
	
	<root>
      <appender-ref ref="asyncAdoNet" />
    </root>

###AsyncRollingFileAppender

	<appender name="asyncRollingFile" type="Log4Net.Async.AsyncRollingFileAppender,Log4Net.Async">
		...
	</appender>
	
	<root>
      <appender-ref ref="asyncRollingFile" />
    </root>


Notes
=============
If the Log4Net.Async library is only referenced in non-code (eg. a log4net log.config file), then the VS build process will not automatically copy the Log4Net.Async dll to the build folder for projects that reference other projects using Log4Net.Async. This occurs even if Copy Local is set to "True".

To get around this some of our users have created a dummy class whose sole purpose is to extend the RingBuffer class, without actually doing anything or being used anywhere at all.

If anyone has an elegant solution to this please let me know or submit a pull request.
