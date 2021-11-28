# LogReader - Reader

Reads in lines from a text file and converts them to data frames for processing by other components.

## Use Cases

This reader is intended to enable log analysis by reading in formatted log entries and allowing listeners and aggregators to track and process the data.

An additional use case is to enable simple log centralization and aggregation. Logs can be read in and sent to a central database, or otherwise converted to a uniform format for centralized processing.

## Configuration

| Parameter | Choices/Defaults | Comment                                                      |
| --------- | ---------------- | ------------------------------------------------------------ |
| format    | String           | This is a mode-specific specification of the fields in the log entry. |
| mode      | freeform, apache | The `apache` mode will parse the format using the [mod_log](https://httpd.apache.org/docs/2.4/mod/mod_log_config.html) specification. The `freeform` mode allows you to specify a fieldname positionally to match when the reader splits the line on spaces. |
| source    | string           | The log file to read.                                        |

## Notes

When using the `apache` mode, some fields will be converted to numeric or date based on their types. This allows for type-specific processing. See the **Output** section below for more details.



## Output

When using the `apache` mode, the following directives are mapped to a corresponding  type and fieldname:
| Directive | Type | Fieldname |
| :-------:| :-------: | --------- |
| A | TEXT | LocalIP |
| B | NUMBER | ResponseSize |
| C | TEXT | Cookie |
| D | NUMBER | ServerTime |
| H | TEXT | RequestProtocol |
| I | NUMBER | BytesReceived |
| L | TEXT | RequestLogID |
| O | NUMBER | BytesSent |
| P | NUMBER | ProcessID |
| R | TEXT | ResponseHandler |
| S | NUMBER | BytesTransferred |
| T | NUMBER | ResponseTime |
| U | TEXT | RequestedURL |
| V | TEXT | ServerName |
| X | TEXT | ConnectionStatus |
| a | TEXT | ClientIP |
| b | NUMBER | ResponseSizeCLF |
| e | TEXT | ENV |
| f | TEXT | Filename |
| h | TEXT | RemoteHostname |
| i | TEXT | VAR |
| k | NUMBER | KeepaliveCount |
| l | TEXT | RemoteLogName |
| m | TEXT | RequestMethod |
| n | TEXT | NoteVAR |
| o | TEXT | ReplyVAR |
| p | NUMBER | ServerPort |
| q | TEXT | QueryString |
| r | TEXT | RequestLine |
| s | NUMBER | Status |
| t | DATE | Time |
| ti | TEXT | RequestTrailer |
| to | TEXT | ResponseTrailer |
| u | TEXT | RemoteUser |
| v | TEXT | CanonicalServerName |

## Examples

This configuration will read an apache access log file:

```json
"Reader": {
    "class": "LogFileReader", 
    "source": "access.log", 
    "mode": "apache", 
    "format": "%h %l %u %t \\\"%r\\\" %>s %b \\\"%{Referer}i\\\" \\\"%{User-Agent}i\\\" \\\"%{UNIQUE_ID}e\\\""
},
```

## Status

This component is considered stable.

