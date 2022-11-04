# EventTracker- Listener

This listener that keeps track of timestamped records. It counts how often timestamped records occur and plots the summaries of the occurrences on a report. 

The component also supports the tracking of the occurrences of a field value.

## Use Cases

The primary use case for this component is to understand the frequency of events in a system. For example, how often orders are generated in a system.

Tracking the occurrences of a field value also provides information as to the composition of the data. It is possible, for example, to see how many of the security events are coming from each of the systems by tracking the "origin" field on a record.

## Configuration

| Parameter | Choices/Defaults       | Comment                                                      |
| --------- | ---------------------- | ------------------------------------------------------------ |
| timestamp | string                 | The field that contains a parsable date-time field.          |
| track     | string                 | the field containing the value the component is to track.    |
| target    | string                 | The name of the file to write the event report. Valid values can include `stdout` and `stderr` for quicker reporting. |
| Include   | string[]               | An array of regular expressions used to match the values. This results in only matching values being tracked. All other values are ignored. See [Oracle's Java Regex Documentation](https://www.oracle.com/technical-resources/articles/java/regex.html) for details on creating regular expressions. |
| exclude   | string[]               | An array of regular expressions used to exclude values. This results in all values being tracked except for those matching one of these expressions.  See [Oracle's Java Regex Documentation](https://www.oracle.com/technical-resources/articles/java/regex.html) for details on creating regular expressions. |
| limit     | number **Default:** 25 | Limit the report to the top must frequent occurrences.  A value of 10 will result in only the top 10 occurrences being reported. A value of zero implies do not track/print occurrences. |

## Notes

### Include and Exclude
If both `include` and `exclude`  are specified in the configuration, only the values that are matched by the include regular expressions are tracked _**unless**_ they are matched by one of the exclude patterns. The `include` patterns are used to determine the values to be tracked and the `exclued` patterns are used to further refine the set. 

For a contrived example, you might specify `/orders/*` as an include pattern and specify `/orders/web/*` as an exclude pattern to track all orders unless they originated from the web.

## Output

*Description of the results of this components operation. Readers format, Transforms results, Writers output.*

## Examples

*These should be cut and paste candidates.* 

*This configuration will...*

```json
{}
```

## Status

This component is considered stable.
