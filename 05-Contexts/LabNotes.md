# Context

The purpose of the context is to provide a "data domain" shared by multiple components.  Components which share the same context can share data with other components in that context by placing data in that context with a shared identifier. Other components can access these data by their identifier.

There are two contexts in the Coyote DX job: Transform Context which allows all components to share data over the life of the job, and the Transaction Context which contains data relating to the current record transform. There is only one Transform Context per job  and there is a Transaction Context for each record which is read in.

The primary purpose of the `TransformContext` is to enable the sharing of data between Job Components at initialization when they are "opened". The configuration of the components can contain values which can be resolved against the context to retrieve data from the context. For example, the context may define a filename which all components can use in their operation. The file may be specified under the "DataFile" context variable and each of the components may reference "DataFile" in their configurations. Instead of using the literal value of "DataFile", components which are context aware will look in the `TransformContext` for the operational value of that context identifier. If the lookup is successful, the context value is used, if not, the literal value is used.

### Persistent Context
It is possible to persist context data between runs. By specifying a `class` configuration varible in the Context section of the job configuration, the Job will load that class as a `PersistentContext` which itself may read and persist context data in storage. THe most common of which is the `FileContext` which reads in the data from a context file within the job directory and use that data for the Transform Context. When the job is completed the `FileContext serializes the data contained within the context to the file system.

Any transform Data specified in the configuration will over-write this data so editing this file may not guarantee these values will be used in the context. All that is needed to use a persistent context is the following in your configuration file:
```JSON
"Context" : { "class" : "FileContext" },
```
One feature of the Persistent Context is the ability to track the number of times the job has been run. Each time the context is opened (at the beginning of a job run) the `RunCount` value is incremented. It is possible to use this run counter in the naming of output files to ensure separation between runs. Like other values in this context, it is possible to edit this value before the job is run to achieve deterministic results.

When deleting elements from your context configuration, they must be manually removed from the context file in the job directory as those values are persisted.

It is possible to specify your own attributes in the persistent context:

```JSON
"Context" : {
	"class" : "FileContext" },
	"fields" : {
		"filename": "out[#$RunCount|0000000000000#].csv"
	}
},
```
In the above example the context will always have the attribute of "filename" saved in the context. In this example it will use the value of the `RunCount` attribute formatted in a zero-padded string of 14 characters. The first time it runs the "filename" attribute will contain "out00000000000001.csv. When next it runs, the "filename" will be "out00000000000002.csv" and so on.

#### Context Versus Symbols
While both the `Context` and `SymbolTable` hole named value pairs, each are designed for a different purpose and have different data lifecycles. Symbols are highly dynamic and are used to resolve this dynamic data into strings. Templates can be used to generate strings using a `SymbolTable` to resolve variables to string values. Symbols are populated by components at various points in the lifecycle and may not be available until after a component has initialized, run, or terminated.

Context variables are populated before any of the components are initialized and may contain domain-specific objects, not just strings. It is used to pass objects between components at different points in the components and jobs lifecycle.

Developers must keep the lifecycles of the components in mind when using the Context. While it is possible, and a common practice for a component to place objects in the in the context during its operation, their existence is intended for the initialization of other components. A task which populates the context with data only does so after that jobs initialization, therefore none of the other components in that job will have access to that data during their initialization. Tasks (pre-processing) may generate objects in the context for other components to use during the read loop and post-processing.

The best practice is to use symbols for template (string) processing and context variables for domain object transfer.