# Split - Transform

Split a field into multiple fields. The result is a copy of the data, but with the data split across multiple fields named the same as the original field with the position of the data appended to the name.

## Use Cases

Consider the datetime string of `2017-11-02T10:21:32.076-0400` where we want two fields, one with `2017-11-02` and the other with `10:21:32.076`. We would first split on the 'T' then again on the '-' to remove the timezone.

Another use case is where a single field may contain multiple values that should be placed in their own fields. A `RequestLine` field from an HTTP log may contain the method, resource and protocol:

```
GET /index.xml HTTP/1.1
```

In this case, the `RequestLine` field can be split on the space delimiter resulting in three new fields:

* `RequestLine.0` - "GET"
* `RequestLine.1` - "/index.xml"
* `RequestLine.2` - "HTTP/1.1"

The above three new fields can be renamed and used separately to count how many hits "/index.html" received.


## Configuration

| Parameter | Choices/Defaults | Comment                                                      |
| --------- | ---------------- | ------------------------------------------------------------ |
| field     | string           | The name of the field to split                               |
| delimiter | string           | The character determining where the split occurs             |
| condition | string           | A boolean expression that, if it evaluates to true, causes the transform to execute. If the expression evaluates to false, the transform will not occur. No condition results in the transform being executed unconditionally. |

## Notes

Using `Split` results in additional fields being added to the working frame. This can cause the memory requirements to increase. It might be useful to `Remove` unwanted fields to reduce the number of fields processed by components in the CDX job.

It is also recommended to `Rename` fields after they have been created by a `Split` to make subsequent transforms more readable.

## Output

When the split operation occurs, new fields will be created with the name of the original and the index of the result appended to the name delimited with a dot. For example, if a field named `text` is  split on the space character and the field is split into three tokens, each token will be placed in a field named `text.x` where `x` is the zero-based index into the result. The result would be the following fields in the 

## Examples


The following will split the timestamp on the "T" character. If the `timestamp` field contains an ISO8601 formatted date such as `2021-11-22T20:31:48Z`, there will be two new fields, `timestamp.0` and `timestamp.1`. The second  `Split` will create three new fields; `timestamp.1.0`, `timestamp.1.1`, and `timestamp.1.2` containing the hours (20), minutes (31), and seconds (48) respectively.

```json
"Transform": {
    "Split": { "field": "timestamp", "delimiter": "T" },
    "Split": { "field": "timestamp.1", "delimiter": ":" }
}
```


## Status

This component is considered stable

