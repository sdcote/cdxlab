# StaticReader - Reader

This reader creates a single, static data frame to pass through the transform engine.

## Use Cases

This is used to insert a single record into a database, and publish a message on messaging platforms.

This reader is also useful in testing and debugging complex transforms. A crafted record can be sent through a transform to test its operation.

## Configuration

| Parameter | Choices/Defaults | Comment                                                      |
| --------- | ---------------- | ------------------------------------------------------------ |
| fields    | Object           | The "Fields" section specifies order and content of the fields. If no fields are specified, an empty frame will be sent. |
| limit     | Number           | This controls how many copies are sent. The default is 1.    |

## Examples

This configuration will create a DataFrame with 3 fields (JobId, Delay, and Log) and pass it through the transform context:

```json
"Reader" : {
     "class" : "StaticReader",
     "fields" : {
         "JobId": "EB00C166-9972-4147-9453-735E7EB15C60",
         "Delay": 1000,
         "Log": true
     }
 },
```

## Status

This component is considered stable.

