# CreateRecord - Listener

 This inserts the target record into the configured database table.

This listener is executed after the working frame is mapped to the target frame so that any mapping operations are included in the processing of the data. Since it runs after mapping, it mimics a writer. 

Using a listener instead of a Writer allows for finer control of the operation. For example, records can be created when it is known the record does not exist by either a lookup of a field value or state of the context. Compare this to a Writer which will only perform upserts. The trade-off is that there is no batching of transactions, each insert is separate from the others.

Transforms can be used to perform lookups and alter the state of the working frame to enable conditions for the listener to be run.

The structure of the table is a normalized attribute table. This allows any structure to be modeled and enables the table to support different record types. Each key-value pair is assigned a parent to which it belongs.All pairs should have a parent as all attributes belong to a data frame.

## Use Cases

The intended purpose of this component

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

