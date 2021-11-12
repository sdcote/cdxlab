# Overview
There are several basic concepts to on which the design is based. Understanding these will make using the toolkit easier.

The toolkit architecture centers around two basic components a Reader and a Writer. The reader reads data in one format and the passes that data to a writer which writes it to another format. It is that simple.

When a reader reads data, it places it in an intermediary data structure which makes accessing the data uniform between components. This data structure is an "Abstract Data Type" and allows any type of data to be accessed and manipulated in a standardized way.

If all the toolkit did was marshal data from the reader to the writer it would be useful enough, but the toolkit uses a Transform Engine to pass the data to other components before the writer so other processing can occur. This allows data read in to be then filtered, validated, transformed and mapped to a different structure before being written. Now we have a toolkit which can solve complex data exchange problems.

# Transform Engine
The core operational component of the toolkit is the Transform Engine (a.k.a. the engine). It is this component which controls the life cycle and processing flow of data through the various components. The engine is responsible for calling the Readers to read a record from some source, and pass that record through each of the processing stages (i.e. components) until the reader has no more data. The engine then performs any configured post-processing tasks and exits.

Transform Engines are passed a configuration in the form of a (JSON) string which it parses to determine what components are to be loaded and passed data for processing. The configuration string instructs the engine what components to load and how to configure each of those components. The toolkit comes with a Loader class which reads a text file from disk and is used to allow the easy invocation of a variety of complex data exchange jobs from a collection of "job" files. It is a common practice to develop a library of batch jobs which are run as is or tweaked with a simple text editor before being run. The loader class makes it easier to run complex jobs from `cron`, at or another scheduler capable of running simple commands.

# Run Cycle
The engine starts by running a set of pre-processing `Task`s and if there are no critical errors, entering a main loop reading in a record with the `Reader`, passing it through sets of `Filter`, `Validator`, and `Transform` components before passing it through a `Mapper` and one of the configured `Writer` components. After the record is written, the loop cycles through again until there are no more records to be read. The following is the basic flow of the default Transform Engine:

## Phases
When the Transform Engine runs, it goes through a set of phases:

1. **Preload** - Run one or more components to prime the context before the engine runs.
1. **Pre-Processing** - a set of Tasks which are run before the other components are created.
1. **Read** - Read in a record from some source using a particular format or protocol.
1. **Filter** - Remove data which is not to be processed.
1. **Validate** - Run a series of checks on the data which made it past the filters.
1. **Transform** - Modify the data in some way.
1. **Aggregator** - Summarize multiple records into one.
1. **Mapping** - Move the data to a target record format for writing to some target.
1. **Write** - Write the mapped data to some target location in a particular format or protocol.
1. **Post-Processing** - execute a set of Tasks after all the data has been read, processed and written.

The Pre and Post processing tasks allow files to be retrieved to the local system for processing and the results sent to another location. They can also be used to manage the files on the local system and to perform clean-up and housekeeping functions.

Not all phases are required. It is possible to just configure the engine to run some (pre-processing) tasks and be done. Sometimes developers will write their own set of validation rules then read a set of records, running them through validation rules to generate a report on the data. It is possible to have several batch jobs which just move data around your system, renaming files and creating generational backups.

## Reader

The role of a `Reader` in the toolkit is to create a `DataFrame` in the `Transaction Context`. The source of the data is often a database or file but there is no limitation in this regard. `Reader` components can retrieve data from any source.

Consider a `Reader` which listens to a message queue. The `Reader` blocks on the queue until a message is received and once received creates a `DataFrame` representing the message and returns it to the  `Transform Engine`. The message is then passed through the the other components in the engine and processed according to the configuration of the other components. It is even possible to utilize `Listener` components to ensure proper processing and issue a response.

Readers must support two methods; `eof()` and `read(Transaction Context)`. The EOF method returns true when there is no more data available and will signal the `Transform Engine` not to attempt to read any data. Returning "true" will normally result in the exiting of the run loop and post processing to occur.

The `read` method simply reads data from its source and returns a `DataFrame` for the other components to process. The engine takes this `DataFrame` and places it in the "source frame" attribute in the current `Transaction Context`. The `Transaction Context` then creates a working copy (i.e. clone) of the source frame in the context for other components to process. This `DataFrame` is called the "working frame" and is open for other components to modify.

Readers are designed to be simple. All they need to do is signal when there is no more data to read and create DataFrame objects from the data it does read.

## Filter

`Filter` components remove data from the processing stream. If the (working) DataFrame matches the filter, it will be removed from the transaction context which will cause the rest of the processing to be skipped.

Filters are components which are passed the records immediately after reading and are intended to remove data from the transaction context (via the working frame) to reduce processing on the other components.

Since they are run before other components, filters have the chance to see all data in the context and perform processing before any transformation activities occur. This opens the possibilities for filters to perform more than just removing data from the processing stream, they may be used to perform analysis on the data and generate auditing records. Although this is not strictly the purpose of the filter, some have written filters to do far more than their intended purpose. This is not best practice, however and such processing is best performed elsewhere (e.g. Listeners).

If there are no filters defined, then all the records will be passed along the chain.

If there are filters defined, then each of the filters will have the opportunity to run.

It is possible for an early exit from the filters; a filter can cause the engine to exit the sequential loop through the remaining filters. Such is the case with the `Accept` Filter. If a transform context matches its condition, the transaction context is immediately accepted and the remaining filters are processed. The same is true for the `Reject` Filter. Because of this, it is best to place a "catch-all" filter at the bottom of the list to perform a default "acceptance" or "rejection" of the context based on the nature of the data expected.

The following filters section will reject all transaction contexts with a working frame containing a field named "Record Type" with a case insensitive value match of "PO":
```
"Filters": {
  "Accept": { "Condition": "match(Working.Record Type,PO)" },
  "Reject": { }
},
```
If it is known that only two record type values will be encountered (e.g. "PO" and "LN") the same effect can be achieved with the following filter configuration:
```
"Filters": {
  "Reject": { "Condition": "match(Working.Record Type,LN)" },
  "Accept": { }
},
```
The above will accept all records unless it is a record type of "LN". This means any bad data (e.g. record type of "IT") will also be processed.

## Validator

Validators are components which perform tests on the Working Frame within a `Transaction Context` and fire events in `Listeners` when their conditions are not met. This allows the `Transform Engine` to perform data quality checks prior to more expensive processing.

Data validation is performed immediately after `Filters` are run and just before the `Transforms`. When validation rules are broken, the `onValidationFailed( OperationalContext, String )` event is triggered in all the registered `Listeners`. This allows the `Transform Engine` to log errors with the Working Frame.

Each validation rule can be configured to cause the `Transaction Context` to abort with the `halt=true` configuration option. This will cause the `Transform Engine` to skip past the remaining validators. The `Transform Engine` will then fire the `onFrameFalidationFailed( OperationalContext, String )` event then attempt to read the next record. The `halt` switch is intended to reduce processing by skipping validations when it is known the working record is invalid. This is helpful when some validators may perform expensive lookups or calculations which can reduce throughput of the job. The default for `halt` is **false**. This allows the Transform Engine to collect all the errors for failed validation.

It is a good idea to have a `Listener` registered with the job to record working records with fail validation so they can be processed off-line.

## Transform

Transforms are components designed to make changes to the Working Frame within the Transaction Context. Transforms are used to perform look-ups on field values in the Working Frame and replace them with new values. Transforms can also perform simple functions such as formatting dates and numbers, changing the case of a string and encrypting sensitive fields.

Transform component are is called with a reference to the current Working Frame after it has been validated. The `Transform` component is expected to return the reference to the transformed frame.

### Complex Transforms
More complex processing can be performed in a transform.  The nature of transformation is completely within the domain of the transform. One developer has written a transform which enriches the data frame by performing a web service lookup on one of the values and adding the results to the working frame for other transforms to use when processing the frame. (A generic version of this transform will be added to future versions of this toolkit.)

## Mapper

This component is responsible for building the Target Frame out of the Working Frame in a particular Transaction Context. It is the Target Frame which is written by the Writers in a Transform Engine.

Mappers allow the marshaling of data with different naming. This is useful if a field is named one thing in the source system but needs to be name something else in the target system.

The order of fields are also controlled by the mappers. The order specified in the mapper configuration is the resulting order the fields will appear in the target frame.

Not all the fields in the Working Frame need be present in the target frame. Normally only a subset of Working Frame fields are mapped so the Writers only write the required fields.

### Configuration
Mappers normally contain a "source to target" configuration convention where the mapper contains a fields section with a list of name value pairs where the name represents the name of the working field and the value represents the name of the target field. The following is a real life configuration of a mapper used in performance testing a ReST-ful web service where thousands of records were read from a CSV, mapped to the appropriate named field, and written with a `WebServiceWriter`:
```
"Mapper": {
  "fields": {
    "description": "short_description",
    "caller": "caller_id",
    "item": "cmdb_ci",
    "urgency": "urgency",
    "impact": "impact",
    "group": "assignment_group"
  }
}
```
In this configuration, the working frame field of description is mapped to the target frame field short_description resulting in the value of description field being written by the writer with a field name of short_description. It will also be the first of 6 fields written.

The above configuration is for the `DefaultMapper` but should work for any Mapper which extends the `AbstractMapper` class.

## Aggregator

An `Aggregator` groups records and releases a summary record based on some configuration. Its purpose is to group or summarize data.

For example,  if  processing the prices of a product over the last week, it would be easier if each products price would appear together as a group so when a new product is observed, the processing of the previous product can be concluded and resources returned for subsequent processing and not left open for the duration of the transfer job.

If you know the data is sorted by an account number, an aggregator can be used to total all records for the same account and only release one record to the writer representing the total of all records for that account.

Aggregation is a special use case and there are only a few aggregators in the toolkit.

## Writer

There can be zero or more `Writer` components that will write the final record to some target.

`Writers` have one simple purpose; write records to some target location / media. The record is always a clone of the `WorkingFrame` of the current `Transaction Context` unless there is a `Mapper` involved which will map fields of the working frame to the target frame for the writer to use.

Writers can write their frames to anything, files, databases, network sockets or to other frameworks through specialized API calls. There is no reason why a `Writer` could not publish a record on a message queue or a pub-sub topic.

It is possible to have multiple writers in a transform job. The default is to unconditionally write all frames passing through the context, but it is also possible to define a boolean expression to control when a record is written. Multiple `Writer` components enables a "fan-out" and "router" pattern in your jobs.

## Listener
Listeners are notified and passed a reference to the `Transaction Context` at various points in run cycle allowing them to perform processing outside of the pipeline. This allows external transformations to be started (`Readers` triggered by `Listener`s) based on the state of the data in a context. This allows the self-orchestration of complex data transforms if desired. Through the use of `Listener`s, it is possible to create very sophisticated integration solutions beyond the "read-transform-write" use case.

It is through the `Listener` components that external systems can be integrated with the mediation process by providing visibility to the data and its various states as it flows through the transform.

# Abstract Data Types

In general terms, an Abstract Data Type is a construct for interacting with the internal structure of data. There is no domain logic (e.g., a data model or business rules) bound to the data type. Its sole purpose is to provide a uniform way to interact with the underlying data.

The Coyote DX toolkit uses the `DataFrame` library to manipulate an array of bytes which represents an array of typed data fields. It allows the developer to work with abstract collections of typed data in a generic manner. It is a `DataFrame` which represents a record which is read in by a `Reader` and written by a `Writer`. Each field in a `DataFrame` has a name (optional) and type (such as Date, Float, Long, String, etc.) and a value (also optional).

Since `DataFrames` are just a generic data access mechanism, they are used for a variety of tasks. The `DataFrame` is used, for example, as a generic configuration object. It is used to marshal JSON data into an abstract structure (think lightweight document object model) and provide programmatic access to data represented by the JSON string. `DataFrame` is a hierarchical, self-describing, type-aware, binary data type which can be used for a variety of tasks.

# Context
All process is performed with a set of data scoped to the lifecycle of the components and the data.

## Transform Context

The entire process runs in an operational context called a `Transform Context` which allows all the components to share data. When the `Transform Engine` is created, it creates a new `Transform Context` and begins to place a variety of data in it for the components within the engine to share.

The `Transform Context` is shared by all components in the `Transform Engine` and is present when components are created, opened, processed and closed. There is only one `Transform Context` which lasts the life of the engine.

It is possible to interrupt the run cycle by setting the `Transform Context` into an error state. Since the `Transform Context` is checked before each phase, it is possible to abort the run if, for example, pre-processing Tasks fail. Of course, this is all configurable.

## Transaction Context

There is a second operational context known as the `Transaction Context` which only lasts the life of one read-write cycle and allows components to share data relating to the current record being passed through and processed by the components in the engine.

When a `Reader` reads a record, it places the record in the `Transaction Context` for the other components to access. Once the `Writer` writes the record, the `Transaction Context` is deleted by the engine and a new one created for the next cycle through the component pipeline.

There are two (`DataFrame`) records in the `Transaction Context`; the first is the source frame read in by the `Readers`, the second is the working frame which all the components take turns accessing and processing. The source frame should not be modified, its purpose is to represent the original data read in by the `Reader`. This is important since the working frame is expected to change as it is passed through `Filters`, `Transforms`, and `Mappers`.

The Transaction Context contains three versions of the current record, source, working, and target:

* **Source Frame** - This is the record that was read in by the reader as the start of the transaction. It is never altered and serves as a backup copy of the original data if other components need it.
* **Working Frame** - This is a copy of the source frame, but serves as a working copy for all the components to transform and otherwise modify as necessary. It may have more or fewer fields than the source frame as its final state depends on the processing of the other components in the pipeline.
* **Target Frame** - This is a final copy of the working frame that is written by all the writers in the transform. The target frame is created by the `Mapper` and may contain fewer fields than the working frame based on the configuration of the mapper. The mapper may also change to ordering of the fields.

