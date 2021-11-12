# ReadRecord - Listener

This reads a record from a table using a specific field as the key.

Using a listener instead of a Writer allows for finer control of the operation.

Transforms can be used to generate the appropriate key values.

This listener operates at the end of the transaction context, giving all other components a chance to process the working frame.

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

