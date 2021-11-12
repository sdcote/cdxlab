# Coyote Lab


## 01 Basics
Go over the basics of installing and calling Coyote.


## 02 Simple
Using the installed version
Show the directory structure
Describe app.home and app.work
Describe the work directory
Describe the job directory - Partitions your data by job as well as job instance.

## 03 Logging 
Introduce the concept of category logging
Loggers are global, for the entire JVM/JRE. Jobs do not have dedicated logging.

## 04 Listeners 
Data profiler
Run with `-owd` to get the results locally
Enable stream analytics, listeners see everything passing through the process and can report in real time the status of the data set flowing through the engine.

## 05 Context 
Shared variables and persistent contexts

## 06 Tasks 
Pre and Post processing for data transfer jobs
Describe the Task job and the concept of 
Clean - Delete the work directory and all the containe job directories

## 07 Repeat 
Jobs will run repeatedly until Ctrl-C is pressed
This is a easy way to perform load testing; use CoyoteDF reader and a writer to your app.
Repeat is also a good way to poll a system from the command line. Just use a reader appropriate for the poll-able interface.

## 08 WebGet 
Downloading files for processing
Show examples of downloading data for processing
Retrieving JSON from a web service with a simple HTTP GET

## 09 Transforms

## 10 Data Integrity 
Shows how to check the integrity of downloaded files

## 11 Service 
Allows multiple jobs to run continually in a sinle JVM/JRE
Each Job contains a "schedule" which dictates when it is to run
Pattern is a simple CronTab entry pattern - Minutes Hours DayOfMonth Month DayOfWeek
Each field in the pattern can be specified as a separate JSON field. All others are defaulted to "any"
Services listen to HTTP requests. A "Manager" runs allowing the service to be controlled.
HTTP Managers can be secured by ACL, DoS filter, SSL/TLS, user accounts/roles.
/api/cmd/shutdown
/api/health
CoyoteUI is an optional project to place a "real" web interface on a service.

## 12 RunJob
Demonstrates how to run groups of jobs with conditional checks
Multiple jobs containing RunJob tasks implement branching
Partition data and then run
Can create complex acyclic graph job flows
Just because you can does not mean you should...these can be difficult to manage

## 13-CmdLineArgs
command line arguments are placed in the context for use in the configurations and component runtimes

## 14-HttpReader

## 15-MessageQueues

## 16-Database

WebService - Retrieving JSON data with a 


## 19-Docker


## 20-ServiceNow

## 21-WebServices

## 25-JsonReader

## 26-Vault

## 99-BitcoinTicker
You will need the CX module added to your installation