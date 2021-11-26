# Rename- Transform

Rename a field in the working copy.

## Use Cases

The primary reason this transform exists is to name the written fields as desired without using a mapper.

Another common use case is to rename the results of a `Split` transform into more meaningful output.

## Configuration

| Parameter | Choices/Defaults | Comment                                                      |
| --------- | ---------------- | ------------------------------------------------------------ |
| source    | string           | The name of the field to rename.                             |
| target    | string           | The target name of the field                                 |
| field     | string           | This is an alias for the "target" parameter.                 |
| condition | string           | A boolean expression that, if it evaluates to true, causes the transform to execute. If the expression evaluates to false, the transform will not occur. No condition results in the transform being executed unconditionally. |

## Notes

The source field will no longer exist once this operation completes. If this is not desired, consider using the `Copy` transform.

The location (field order) of the field will not change. If you are renaming the third field in the record, the renamed field will be in the third position.

## Output

The named field (source) will have it name changed. No other alteration to the field or record will occur.

## Examples

The syntax is relatively simple:

```json
"Transform": {
    "Rename": { "source": "oldField", "field":"newField"}
}
```

## Status

This component is considered stable

