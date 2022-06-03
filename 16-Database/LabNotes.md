# Database

The Coyote DB module adds the ability to work with JDBC databases. It is not part of the core toolkit and must be installed separately.

There are two flavors in this lab, one for H2, a fully embedded database allowing you to run the examples locally, and one for Oracle. The Oracle example assumes you have an Oracle database service running somewhere.

**NOTE:** Some URL interpretations will not work on Windows hosts since Windows uses the traditional escape character in its paths. Therefore `jar:file:[#$user.dir#]/h2-1.4.196.jar!/` will convert into something like `jar:file:/home/juser/workspace/h2-1.4.196.jar!/` on Linux hosts, but something like `jar:file:C:\Users\juser\workspace/h2-1.4.196.jar!/` on Windows boxes which is an invalid URL. Use a valid, fully-qualified URL for your host e.g., `jar:file:///C:/Users/juser/workspace/h2-1.4.196.jar!/`, or place your JAR in the CDX `lib` directory.

# JdbcWriter

run the example that will write the CSV file to a database



# JdbcReader

run the example that read the record from the database

```shell
cdx -owd 
```



# Database Fixtures

There is a task that places the database in the context of other database-oriented components to share.



