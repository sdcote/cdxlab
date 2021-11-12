# Overview

It is possible to run data transfer jobs from within other jobs. This capability, combined with shared data in Transform Contexts and conditional processing allow data transfer jobs to be chained together to perform complex processing and job orchestration.



# RunJob Task

The `RunJob` task simply runs a named job as a part of pre or post processing.

# RunJob Listener

The `RunJob` listener runs another job when for every record that is read. The contents of the record is made available to the called job via its symbol table. This allows the called job to use templates in its configuration to alter its operation.

A common use case is reading in a list of arguments for several job executions. Consider a web service that generates performance metrics for applications. A data transfer job reads from that web service performance metrics for an application and records those metrics in a database for later trend analysis. Normally a data transfer job would be written for each application specifying the parameters for the web service call and the target of the metrics. Instead of hundreds of jobs being written with their different parameters, a single job can be specified and the parameters read in by a reader. The RunJob Listener would then call a templated job and use the data from the current transaction to satisfy the templated parameters in that job. Instead of hundreds of job files, there is only one. The data read in will determine how many jobs are run.

The `RunJob` listener is called when the `onMap()` event is triggered. The listener then runs the job specified in it's `filename` configuration property. A new transform engine is created and the fields of the target frame are placed in the new engine's symbol table along with merging all the symbols of the calling job. The called job is then executed normally.

All the results of the called job are expected to be placed in the jobs working directory. Nothing is passed back to the calling job. This listener is intended to only call multiple instances of the same job and use the target frame as the means to communicate parameters to the called jobs.

```
cdx -owd RunJobListener.json
```
The above reads in data from the `jobdata.csv` file and calls the `WeatherForecast.json` job several times; once for each record in the CSV file. 

Looking in the `WeatherForecast.json` file will reveal several template variables that correspond to the fields in the CSV file. The different values in the CSV file control how the called job operates.





