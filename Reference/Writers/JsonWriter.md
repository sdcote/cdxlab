# JsonWriter

Writes a data frame as a simple JSON string to either standard output (default), standard error, or any other file.
## Use Cases

This writer is used to convert processed frames into a standard JSON format for other components to process.

## Configuration

| Parameter | Choices/Defaults              | Comment                                                      |
| --------- | ----------------------------- | ------------------------------------------------------------ |
| append    | true or **false** | True = Append the data frames to the existing file. False(default) = Overwrite the contents of the existing file with the frames in this run. |
| enabled   | **true** or false                          | True(default) = the writer will write received target frames. False = the frames will not be written. |
| target | **stdout**, stderr, or File URI, File Name | stdout(default) - the frames will be written to the standard output device (console), stderr - the frames will be written to the standard error device. If a file URI is provided, the frames will be written to that file. If a filename is used, it will be converted to a file URI. Relative file names will be placed in the job directory. |

## Notes

Data is written in UTF-8 encoding.

### Troubleshooting

The file URI must contain at least 2 slashes (`file://`) to be considered valid. If only 2 slashes are used, the file is considered relative. Absolute file URI contain 3 slashes (`file:///`) with the third slash representing the root. For example:

`file://test.txt` is relative but `file:///test.txt` resides in the root (i.e. `/test.txt`).

## Output

The output is a formatted JSON with each field on a separate line for readability.

## Examples

This configuration will write formatted JSON to the console. This is useful during the development and debugging of data transfer jobs:

```json
"Writer": { "Class": "JsonWriter" }
```

This configuration disables the writer. If `enabled` is set to true, it would write formatted JSON to the console:

```json
"Writer": { "Class": "JsonWriter", "enabled": false }
```

This configuration will write formatted JSON to the `Metric.json` file in the job directory appending frames from this job run with those of past runs:

```json
"Writer": { "Class": "JsonWriter", "target": "Metrics.json", "append": true }
```

## Status

This component is considered stable

