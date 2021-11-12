# UpdateRecord - Listener

This updates the configured database table based on the values in the working record.

Using a listener instead of a Writer allows for finer control of the operation. For example, records can be updated when it is known the record does already exists by either a lookup of a field value or state of  the context. Compare this to a Writer which will only perform upserts.

Transforms can be used to perform lookups and alter the state of the working frame to enable conditions for the listener to be run.

This listener operates at the end of the transaction context, giving all other components a chance to process the working frame and the mapper to generate a properly formatted record for updating in the database.

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

