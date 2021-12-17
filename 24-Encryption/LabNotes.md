# Encryption

The CDX toolkit contains a light-weight internal encryption facility that allow you to place secrets in your configuration files. The master key is provided at runtime so there is no reason to expose sensitive data in your configurations.



The primary approach is to set an environment variable that contains the master key and optionally the encryption strategy.

Consider updating the cdx scripts to include your master key by default. This will use the same key for all the jobs run on the host.



Configurations support the `keyfile` parameter that will read the master key from a (protected) file on your disk. This operates similar to other "vault" features. You can place the master key in this file (normally in your home directory) so you can check-in your CDX configurations to a source repository with the encrypted secrets. Other users will just need to place the same file in their environments to run the CDX jobs.



```
cdx encrypt text
```

```
cdx encrypt text somekey
```

```
cdx encrypt text somekey XTEA
```

