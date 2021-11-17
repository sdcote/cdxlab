# DatabaseContext - Context

This is an operational context backed by a database which allows values to be persisted on remote systems.

The data in the context can be managed externally without regard to where the transform is being run, allowing transforms to be run on many different hosts without having to manage locally persisted context data.

Key value pairs specified in the fields section are used to reset the field values in the database context at the start of the job. Their values will be persisted when the context is closed. Other jobs using the context will then have access to these values unless they reset them in a similar fashion.

Unlike a writer, this component deals with fields of a dataframe not the dataframe itself. each field is a record in the table differentiated by the field name and the name of the job to which it belongs.

## Use Cases

The primary use case is the running of transform jobs in a pool of distributed instances in the cloud. A particular instance will run one on one host and run another time on a different host. 

Another use case is running jobs in virtual machines with ephemeral file systems such as Docker. The compute instance is restarted at least daily with a fresh file system and all local files are lost. This context allows persistent data to be stored remotely so that local file access is not required.


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

