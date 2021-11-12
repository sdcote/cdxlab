# DatabaseFixture - Task

Places a database configuration in the context for other database-oriented components to retrieve.

## Use Cases

There are times when the same database connection information is shared by multiple components in the configuration. This allows the  database configuration to be defined once and referenced by other components, allowing the configuration to the managed in one location.

## Configuration

| Parameter    | Choices/Defaults    | Comment                                                      |
| ------------ | ------------------- | ------------------------------------------------------------ |
| name         | **Required** String | This is the name other components will use to address this fixture. |
| target       |                     |                                                              |
| driver       |                     |                                                              |
| username     |                     |                                                              |
| password     |                     |                                                              |
| ENC:username |                     |                                                              |
| ENC:password |                     |                                                              |
| library      | String              | This is a link to the JAR that contains the JDBC driver      |

## Notes

Anything the user should know about operation including troubleshooting and implications.

## Output

This task will create a database fixture object and place it in the transform context using it name as its access key.

## Examples

These should be cut and paste candidates. 

This configuration will...

```json
"DatabaseFixture": { 
    "name": "H2", 
    "target": "jdbc:h2:./staging", 
    "driver": "org.h2.jdbcx.JdbcDataSource", 
    "username": "sa",
    "password": "5up34seC43T"
}

```

## Status

This component is considered stable.

