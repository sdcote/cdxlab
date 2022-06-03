# Readers

The role of a Reader in the CDX toolkit is to create a DataFrame in the Transaction Context. The source of the data is often a database or file but there is no limitation in this regard. Readers can retrieve data from any source using a blocking model if desired.

Consider a Reader which listens to a message queue. The reader blocks on the queue until a message is received and once received creates a DataFrame representing the message and returns it to the caller (the Transform Engine). The message is then passed through the the other components in the engine and processed according to the configuration of the other components. It is even possible to utilize Listeners to ensure proper processing and issue a response.

Readers must support two methods; `eof()` and `read(Transaction Context)`. The EOF method returns true when there is no more data available and will signal the Transform Engine not to attempt to read any data. Returning false will normally result in the exiting of the run loop and post processing to occur.

The read method simply reads data from its source and returns a DataFrame for the other components to process. The engine takes this DataFrame and places it in the "source frame" attribute in the current Transaction Context. The Transaction Context then creates a working copy (i.e. clone) of the source frame in the context for other components to process. This DataFrame is called the "working frame" and is open for other components to modify.

Readers are designed to be simple. All they need to do is signal when there is no more data to read and create DataFrame objects from the data it does read.

## Core Project

* **ContextReader** - retrieves its DataFrames from its transform context. (See Context Writer)

* **CsvReader** - Read Character Separated Value (CSV) files.
* **FlatFileReader** - Reads frames from fixed-length field format as found in EDI and Mainframe environments.
* **HttpReader** - Listens for HTTP requests on a particular port and converts those requests into DataFrames of processing (e.g. Web Service)
* **JsonReader** - Read JSON files.
* **LogFileReader** - parses each line into tokens and maps those tokens to fields based on the format and parsing mode.
* **NullReader** - A do-nothing implementation of a reader, useful for testing.
* **StaticReader** - Generates a single record to send through the engine.
* **XmlReader** - Read XML files.

## Coyote DB

* **JdbcReader** - Executes SQL queries and generates records from the result set.

## Coyote WS

* **WebServiceReader** - Makes a request of a remote web service and uses the response for data records.

# Writers

CDX Writers have one simple purpose; write DataFrames to some target location / media. This DataFrame is always a clone of the working frame of the current Transaction Context unless there is a Mapper involved which will map fields of the working frame to the target frame for the writer to use..

Writers can write their frames to anything, files, databases, network sockets or to other frameworks through specializes API calls. There is no reason why a writer could not publish a DataFrame on a message queue or a pub-sub topic.

It is possible to have multiple writers in a transform job. The default to unconditionally writing all frames passing through the context, but it is also possible to define a boolean expression to control when a frame is written. This enables a "fan-out" and "router" pattern in your jobs.

## Core Project

* **ConsoleWriter** - Writes frames to STDOUT in JSON, CSV,or XML.
* **ContextWriter** - Write frames to a context variable as an array of frames. (See ContextReader)
* **CsvWriter** - Writes frames to a CSV file.
* **FlatFileWriter** - Writes frames into fixed-length field format as found in EDI and Mainframe environments.
* **JsonWriter** - Writes a data frame as a simple JSON string to either standard output (default) orany other file target.
* **NullWriter** - A do-nothing implementation of a writer useful for testing.
* **PropertyWriter** - Writes a data frame as a simple name-value pair string to either standard output (default) or any other file target.
* **XmlWriter** - Writes a data frame as a simple XML string to either standard output (default) or any other file target.

## Coyote DB

* **JdbcWriter** - Writes frames into a database, each field in its corresponding named column.

# Filters

CDX Filters remove data from the processing stream. If the (working) DataFrame matches the filter, it will be removed from the transaction context which will cause the rest of the processing to be skipped.

Filters are components which are passed the records immediately after reading and are intended to remove data from the transaction context (via the working frame) to reduce processing on the other components.

Since they are run before other components, filters have the chance to see all data in the context and perform processing before any transformation activities occur. This opens the possibilities for filters to perform more than just removing data from the processing stream, they may be used to perform analysis on the data and generate auditing records. Although this is not strictly the purpose of the filter, some have written filters to do far more than their intended purpose. This is not best practice, however and such processing is best performed elsewhere (e.g. Listeners).

If there are no filters defined, then all the records will be passed along the chain.

If there are filters defined, then each of the filters will have the opportunity to run.

It is possible for an early exit from the filters; a filter can cause the engine to exit the sequential loop through the remaining filters. Such is the case with the Accept Filter. If a transform context matches its condition, the transaction context is immediately accepted and the remaining filters are processed. The same is true for the Reject Filter. Because of this, it is best to place a "catch-all" filter at the bottom of the list to perform a default "acceptance" or "rejection" of the context based on the nature of the data expected.

The following filters section will reject all transaction contexts with a working frame containing a field named "Record Type" with a case insensitive value match of "PO":

    "Filters": {
      "Accept": { "Condition": "match(Working.Record Type,PO)" },
      "Reject": { }
    },

If it is known that only two record type values will be encountered (e.g. "PO" and "LN") the same effect can be achieved with the following filter configuration:

    "Filters": {
      "Reject": { "Condition": "match(Working.Record Type,LN)" },
      "Accept": { }
    },

The above will accept all records unless it is a record type of "LN". This means any bad data (e.g. record type of "IT") will also be processed.

## Core Project

* **Accept** - Accept the record if it matches the given condition. The absence of a condition is an unqualified match.

* **Reject** - Reject the record if it matches the given condition. The absence of a condition is an unqualified match.

# Tasks

CDX Tasks are simple units of functionality that execute before any DataFrames are read and after all read-write transactions are complete. Their purpose is to initialize and clean up the working environment before and after the transformation process runs. They are designed to operate outside of the read-write cycle.

The original purpose of the tasks was to retrieve files from file shares into the working directory to the readers could process those files. The results of processing (what the writers wrote) were then moved to their proper file locations for later retrieval.

Tasks can be created (via Java coding) to perform any function needed, so the toolkit is very open for extension.

If processing is to occur within the read-write cycle, the developer can create a Listener to perform processing when events occur when a data frame is read and written.

## Core Project

* **Archive** - generates a compressed archive of a file or directory. (i.e., create a zip file)
* **Backup** - Make a copy of a file in the given file using a generation naming scheme effectively rotating the files.
* **CheckAlder32** - Perform a Adler32 checksum on the given file and compare it with the checksum in a file with the same name with a ".adler32" suffix.
* **CheckCRC** - Perform a CRC32 checksum on the given file and compare it with the checksum in a file with the same name with a ".crc32" suffix.
* **CheckMD5** - Perform a MD5 checksum on the given file and compare it with the checksum in a file with the same name with an ".MD5" or an ".md5" suffix.
* **CheckSHA1** - Perform a SHA-1 digest on the given file and compare it with the checksum in a file with the same name with a ".sha1" suffix.
* **CheckSHA256** - Perform a SHA-256 checksum on the given file and compare it with the checksum
   * in a file with the same name with a ".sha256" suffix.
* **CheckSize** - Check the size of the given file.
* **CheckSymbolNotNull** - abort a job if a symbol is missing from the symbol table of the transformation context.
* **Clear** - Clear out the contents of a directory or delete it altogether.
* **Combine** - Combine all the text files into one.
* **Copy** - Make a copy of the source file in the target location.
* **Delete** - Delete a file or a directory from the file system.
* **FileJob** - searches for a set of files in a directory and runs a job with the filename set in the context.
* **LogContext** - Log the current state of the context including the symbol table.
* **LogEntry** - generates a log entry.
* **LogSymbols** - Log the current state of the symbol table.
* **Message** - generates a console message.
* **Move** - Move the source file to a target location.
* **Process** - Runs a process on the host operating system.
* **ReadIntoContext** - Read a text file into the context as a set of name-value pairs.
* **RunJob** - runs a data transfer job using the current context.
* **SaveContextValue** - Save a context variable to a file.
* **SetSymbol** - sets a value in the symbol table with either a static value or the result of an evaluation.
* **Sleep** - Sleep for the given number of milliseconds.
* **Touch** - Update the last access time of a file.
* **Unzip** - Unzip the given file.
* **WaitForFile** - wait for a file to arrive and to be readable.
* **WebGet** - Retrieve the data via the given source URL and place it in the job directory unless a file or directory is specified.
* **Zip** - This task is an alias/synonym for the Archive task.

## Coyote DB

* **DatabaseFixture** - Create a database fixture in the Transform context other components can share.
* **RunSql** - Runs a set of SQL commands from a file against a database connection.
* **TableLoad** - Wraps a FrameReader (CSV, JSON, or XML) and a JdbcWriter together to perform the loading of a file into a database.

# Validators

CDX Validators are components that perform tests on the Working Frame within a Transaction Context and fire events in Listeners when their conditions are not met. This allows the Transform Engine to perform data quality checks before more expensive processing.

Data validation is performed immediately after Filters are run and just before the Transforms. When validation rules are broken, the `onValidationFailed( OperationalContext, String )` event is triggered in all the registered Listeners. This allows the Transform Engine to log errors with the Working Frame.

Each validation rule can be configured to cause the Transaction Context to abort with the `halt=true` configuration option. This will cause the Transform Engine to skip past the remaining validators. The Transform Engine will then fire the `onFrameFalidationFailed( OperationalContext, String )` event and then attempt to read the next record. The `halt` switch is intended to reduce processing by skipping validations when it is known the working record is invalid. This is helpful when some validators may perform expensive lookups or calculations which can reduce the throughput of the job. The default for the `halt` parameter is `false`. This allows the Transform Engine to collect all the errors for failed validation.

It is a good idea to have a `listener` registered with the job to record working records that fail validation so they can be processed offline.

## Core Project

# Listeners

CDX Listeners are notified by the Transform Engine when events occur during the run cycle. One or more listeners can be registered with the Transform Engine to perform processing at specific points in the process.

A common listener simply logs when validation errors occur so problems with data quality can be recorded. Other listeners track the beginning and end of the Transaction Context and the Transform Context to determine the throughput for each transaction and the job overall.

When an event occurs, each of the listeners is notified of the context in which the event occurred so the Listener has all the data and references it needs to perform its processing.

Events and Listeners are a great way to extend the toolkit to perform complex processing specific to your needs.

## Core Project

- **ContextDumper** - records the data in the context at the beginning and the end of a transformation.
- **ContextLogger** - logs all events to a file for the batch.
- **DataProfiler** - keeps track of the data read in to and out of the engine and reports on the characteristics of the data observed.
- **EventProfiler** -  tracks the occurrences of a tracked field providing a variety of information about the field.
- **FieldTotal** - Track the running total for a particular column and place the result in both the context and the symbol table for other components to use.
- **PercentChange** - compares the current frame against the last frame cached from a subsequent read/mapping event or the average of all previously cached frames.
- **RunJob** - execute the named job when after all the fields of the working frame have been mapped to the target frame.
- **Validation** -  writes the record with all the errors for later processing.



## Coyote DB

* **CreateRecord** - inserts the target record into the configured database table.
* **DataProfilerSql** - tracks the data read in to and out of the engine and reports on the characteristics of the data observed including the SQL required for creating tables to hold the data read in and written out.
* **DeleteRecord** - deletes a record from a table using a specific field as the key.
* **ReadRecord** - reads a record from a table using a specific field as the key.
* **UpdateRecord** - updates the configured database table based on the values in the working record.

# Transforms

CDX Transforms are components designed to make changes to the Working Frame within the Transaction Context. Transforms are used to perform lookups on field values in the Working Frame and replace them with new values.

Transforms can also perform simple functions such as formatting dates and numbers, changing the case of a string, and encrypting sensitive fields.

## Core Project

- **AlertManager2Markdown** - This is an example of a custom transform. It converts a DataFrame received from an Alertmanager webhook call into a frame format suitable for sending to Cisco WebEx or MicrosoftTeams.
- **Append** - appends to the value of a field with a particular value based on some condition.
- **Copy** - Copy one field to another.
- **Counter** - Place a sequential number in the named field.
- **Date** - Create a native date object from the data existing in the field.
- **Format** - Alias for the Text transform. (*under development*)
- **Guid** - Place a random globally unique identifier (GUID) in the named field.
- **KeepOnly** - Removes all fields from the frame except for the named field.
- **Multiply** - Multiply a field by a factor.
- **Numeric** - set a number in a named field. (*under development*)
- **Remove** - Remove a field.
- **Rename** - Rename a field.
- **Replace** -  replace the target string with the value in the given named field.
- **Set** - sets the value of a field to a particular value based on some condition.
- **Split** - Split a field into multiple fields.
- **Subtract** - subtract one field from another or use literals if desired.
- **Text** - Create a text (String) object from the data existing in the field.
- **Timestamp** - Places the current date-time in the configured field.



# Mappers

This component is responsible for building the target frame out of the Working Frame in a particular Transaction Context. It is the target frame that is written by the Writers in a Transform Engine.

Mappers allow the marshaling of data with different naming. This is useful if a field is named one thing in the source system but needs to be name something else in the target system.

The order of fields is also controlled by the mappers. The order specified in the mapper configuration is the resulting order in the fields that will appear in the target frame.

Not all the fields in the Working Frame need to be present in the target frame. Normally only a subset of Working Frame fields is mapped so the Writers only write the required fields.

## Core Project

# Context

## Core Project

## Coyote DB

* **DatabaseContext** - creates an operational context backed by a database that allows values to be persisted on remote systems.
