# CheckSymbolNotNull - Task

This task checks to see if a symbol in the transformation context symbol table  exists and contains a value.  If the value does not exist, the task aborts the job by throwing a `TaskException`.

An empty ("")  or blank (i.e. all whitespace) value will also cause an exception to be thrown.

## Use Cases

The original purpose of this task is to abort a job if a critical symbol used in subsequent processing is missing from the job. This can prevent expensive processing from occurring if a required variable provided by the symbol table is missing or empty.

## Configuration

| Parameter   | Choices/Defaults                | Comment                                                      |
| ----------- | ------------------------------- | ------------------------------------------------------------ |
| symbol | String           | This specifies the name of the symbol to check. It is required and will throw a `ConfigurationException` if not specified. |
| condition | String | This is a boolean expression that controls if the task is fired. |

## Notes

This task ignores the `haltOnError` configuration parameter normally supported by tasks since its purpose is to halt processing of the job.

## Output

The `TaskException` is thrown if the named symbol is missing from the symbol table of the transformation context.

## Examples

In this example, we check if the `INCIDENT_URL` symbol exists. If it does not, there is no need to continue processing of the job.
```json
"CheckSymbolNotNull": {
  "symbol": "INCIDENT_URL"
}
```

This is the same example, but will only check the symbol if the current hour (`HH` symbol) is greater than 8. So only check after 8am.
```json
"CheckSymbolNotNull": {
  "symbol": "INCIDENT_URL",
  "condition": "HH > 8"  
}
```

## Status

This component is considered stable

