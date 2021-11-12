# Logging

Logging is performed at the runtime level. This means loggers receive events from all components in the runtime.

Loggers may be ephemeral, active only for a short period of time, but while they are active they will receive events from all components in the runtime. This is both a good thing as the logs will contain events that may affect operation and bad as they may contain more events than expected causing some confusion during debugging.

## Global Logging

Loggers configured at the top level component are initialized when the top level component is created. These loggers are terminated when the runtime is terminated. Due to this lifecycle, the symbol table available for template resolution is limited to system properties. This is normally not an issue, but this causes the logging environment (e.g. template resolution) to be unaware of job-specific values.

This lifecycle also implies all events from all jobs and all components will be captured. This can lead to rather large log streams.

## Job Logging

Jobs may contain their own logging section. This results in a set of loggers that will have access to the Transform Context when being initialized allowing enhanced symbols to be resolved in configuration templates. Loggers can therefore be initialized with the job directory populated in the symbol table and templating can be set with the target of the logs within the job directory.

Lob loggers are also terminated when the job terminates. This allows the loggers to be active only when the job is active narrowing the scope of events captured. If a Job in a Service runs at midnight, the lob loggers will only capture the events that occurred while the job was active. Consider a Service containing dozens of jobs running throughout the day. With Job Logging, only the events issued while a job was active is recorded and placed in the job directory to make trouble shooting easier.

Of course, any global loggers configured will still be active and capture everything during the life of the runtime.



# Job Logger

Notice the two  `CyclingFileAppender`loggers in the job configuration. The "Debug" configuration used the `[$#jobdir#]`template symbol to specify the path to the `Debug.log`file.  Normally this symbol would not exist until after the job initializes and cannot be used in the global logger. The "Error" log has a relative file name which will automatically be placed in the job directory since it is a job logger. Both resolve to the same directory.