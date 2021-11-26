# RunSql - Task

This runs a set of SQL commands from a file against a database connection.

With this task, it is possible to execute a SQL script against a JDBC connection to perform complex database processing. 

## Use Cases

The goal is to fully customize a relational database for subsequent use by other components in  the job.

## Configuration

| Parameter | Choices/Defaults                       | Comment                                                      |
| --------- | -------------------------------------- | ------------------------------------------------------------ |
| source    | **Default:** true                      | This is a long description of how the parameter affects operation. |
| mode      | apache, freeform **Default:** freeform | This configures how the format configuration parameter is parsed. |
| format    | String                                 | This is a string describing how to parse each column of the log entry |

## Notes

Anything the user should know about operation including troubleshooting and implications.

### Apache
The apache format described in the apace documentation is fully supported.

### Freeform
The freeform mode parses the name and matches it to the same column in the log entry field.

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

