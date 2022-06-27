# LdapReader - Reader

This is a frame reader which queries directory services via the LDAP protocol to generate frames for located records.


## Use Cases

The intended purpose of this component

## Configuration

| Parameter    | Choices/Defaults | Comment                                                      |
| ------------ | ---------------- | ------------------------------------------------------------ |
| class        | **JdbcReader**   | This is the name of the class to load. It must be `JdbcReader`. |
| source       | String           | This references a server name and optional port to which the reader will connect.. |
| query        | String           | This is the SQL that will be used to retrieve records.       |
| name         | String           | This is the name of the entries to retrieve                  |
| filter       | String           | This is an optional specification of a filter to match to help in narrowing the result set to specific entries. If omitted, a default of `(objectClass=*)` will be used. |
| fields       | String[]         | This is an array of attribute names to retrieve from the entry that will be placed in like-named fields in the data frame. |
| username     | String           | The username to use when making the connection.              |
| password     | String           | The password to use when making the connection.              |
| ENC:username | String           | The encrypted username to use when making the connection.    |
| ENC:password | String           | The encrypted password to use when making the connection.    |

## Notes

When using encrypted values for username or password, be sure to properly specify the secret key used to encrypt the configuration values. See the Encryption lab notes for more information on the encryption capabilities of the toolkit. Additionally, a vault can be used to provide secrets.

## Output

For each entry located, a data frame will be passed through the transformation engine for processing.

## Examples

This configuration connects to a directory service at `ldapserver.example.com` with credentials. It then searches for all entries that match the `objectClass=group` filter at `OU=Security Groups,DC=corp,DC=example,DC=com` and returns the `cn`, `description`, `displayName`, `info`, and `name` attributes:

```json
"Reader" : {
  "class" : "LdapReader",
  "source" : "ldapserver.example.com",
  "username": "juser",
  "password": "53c4Et",
  "name": "OU=Security Groups,DC=corp,DC=example,DC=com",
  "filter": "(objectClass=group)",
  "fields": [
    "cn",
    "description",
    "displayName",
    "info",
    "name"
  ]
},
```

## Status

This component is currently under development.

