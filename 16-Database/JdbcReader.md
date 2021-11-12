# JdbcReader

This is a frame reader which uses a JDBC result set to create frames.


## Use Cases

The intended purpose of this component

## Configuration

| Parameter    | Choices/Defaults | Comment                                                      |
| ------------ | ---------------- | ------------------------------------------------------------ |
| class        | **JdbcReader**   | This is the name of the class to load. It must be `JdbcReader`. |
| source       | String           | This references a named [DatabaseFixture](DatabaseFixture.md) in the configuration. If other databse connection attributes are used, this will cause the fixture to be used instead of loading a driver with the other configuration options. |
| query        | String           | This is the SQL that will be used to retrieve records.       |
| driver       |                  |                                                              |
| library      | String           | This is an optional specification of the JAR to use when looking for the driver.  This parameter is not required if the driver can be found in the classpath. |
| username     | String           | The username to use when making the connection.              |
| password     | String           | The password to use when making the connection.              |
| ENC:username | String           | The encrypted username to use when making the connection.    |
| ENC:password | String           | The encrypted password to use when making the connection.    |

## Notes

When using encrypted values for username or password, be sure to properly specify the secret key used to encrypt the configuration values. See the Encryption lab notes for more information on the encryption capabilities of the toolkit.

## Output

For each record that is returned, a data frame will be passed through the transformation engine for processing.

## Examples

This configuration used a [DatabaseFixture](DatabaseFixture.md) with the name of "H2": 

```json
"Reader": {
    "class": "JdbcReader",
    "source": "CDB",
    "query" : "select * from EMPLOYEE_VIEW"
}
```
This configuration specifies all the database configuration options within the reader:

```json
"Reader" : { 
    "class" : "JdbcReader",
    "source" : "jdbc:h2:[#$user.dir#]/userdb",
    "driver" : "org.h2.Driver",
    "library" : "jar:file:[#$user.dir#]/h2-1.4.196.jar!/",
    "username" : "sa",
    "password" : "",
    "query" : "select Role, Username, License from sa.user"
}
```

## Status

This component is considered stable

