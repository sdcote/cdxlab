# JdbcWriter

Writes records to a relational database via JDBC connection

## Use Cases

The intended purpose of this component

## Configuration

| Parameter | Choices/Defaults  | Comment                                                      |
| --------- | ----------------- | ------------------------------------------------------------ |
| Name      | **Default:** true | This is a long description of how the parameter affects operation. |

## Notes

There is an inherent loss of fidelity with this class in that not all SQL types are replicated faithfully through the writer. For example, the differences between VARCHAR, VARCHAR2, STRING are merged into a String type.

NUMERIC and LONG are simply the Long type. The goal of this writer is to support the widest number of types across not only different data bases, but data storage technologies and network protocols. The core intermediate format (DataFrame) is designed to support data types common to many different technologies and therefore dictates what the framework supports.

You will need access to the JDBC driver for the database you are going to use. The JAR can be specified in the configuration and can point to the common location of all your JDBC drivers.

Thin (type 4) drivers are easier to use, but any JDBC driver is acceptable. The important concept is the driver must be in the classpath of the Java runtime. 

Another approach is to place the driver and it's dependencies in the `lib` directory of the Coyote installation directory. This will result in the driver being accessible to all data transfer jobs. The downside to this is you will be locked into one version of that driver. This should not be an issue for most since drivers are usually backward compatible. Just keep this in mind when making your decisions.

### Troubleshooting

The most common error seen is the driver for the database cannot be found:

```
ERROR | Could not connect to database: ClassNotFoundException - oracle.jdbc.driver.OracleDriver
```

This can be solved by specifying the `library` configuration option, or placing the driver in the `lib` directory of the Coyote installation directory.



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

