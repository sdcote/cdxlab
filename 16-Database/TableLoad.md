# TableLoad - Task
This wraps a FrameReader and a JdbcWriter together to perform the loading of a file into a database.

## Use Cases

There are just some things that are best perfomed in a database and through the use of lighweight databases such as H2 and SQLlite it is possible to stage large amounts of data on disk without consuming large amounts of memory. The data can then be used in reader, writers, transforms, and other components without replicating functionality in the framework.

It is possible to create a database for a specific task and then remove it from disk once the job is complete with post-processing tasks. This allows for some rather dynamic processing as CSV files can be generated in other jobs before this job is executed, allowing for the creating of JDBC databases with fresh data. 

## Configuration

| Parameter | Choices/Defaults  | Comment                                                      |
| --------- | ----------------- | ------------------------------------------------------------ |
| Name      | **Default:** true | This is a long description of how the parameter affects operation. |

## Notes

Anything the user should know about operation including troubleshooting and implications.

## Output

Description of the results of this components operation. Readers format, Transforms results, Writers output.

## Examples

These should be cut and paste candidates. 

This configuration will...

```json
{}
```

## Status

This component is considered stable

