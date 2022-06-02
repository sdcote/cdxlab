# Services

Services to not exit; they continue running until they are terminated.

The primary use case for a `Service` is to group together a set of `Job` instances that are to run continually or on some schedule. Starting and stopping the service starts and stops all the Jobs in that service.



# Simple Configuration

The simplest configuration is arguably a service that runs a single `Job`. To demonstrate this, run `SimpleService.json`:

```
cdx -owd SimpleService.json
```

You will notice that is will run the job, but the process will not return to the command line.

```
INFO | Component initialization is complete
INFO | Service initialized - Runtime: 1.8.0_112 (Oracle Corporation) - Platform: amd64 OS: Windows 10 (10.0)
INFO | Loader is operational
```

This service is configured to run a job every 5 minutes. Once the time has arrived, the Job will run and then return to waiting for the next time to run. The service will remain running.

To terminate the service, press `Ctrl-c` and the Java Runtime will terminate. Services utilize shutdown hooks with the runtime and all components will be terminated gracefully when the runtime terminates.

# Running Services in the Background

All the exercises involve running the services in the foreground.  This is not always the ideal way to run services. It is more likely that once you develop your service and it is ready to regular use, you will want to run it in the background.

## Linux

Running Coyote services in the background in Linux environments is relatively easy:

```
nohup cdx daemon.json > /dev/null 2>&1 & 
```

To capture the PID for later:

```
nohup cdx daemon.json > /dev/null 2>&1 & echo $! > daemon.pid
```

You can then check to see if the service is running with: 
```
ps -aux | grep `cat daemon.pid`
```
and later kill the process with:
```
kill -9 `cat daemon.pid`
```

## Unix Daemon Support

You can use the `inetd` subsystem in Unix environments to keep CDX running in the background even after reboots.

**Note:** This capability requires the LSB package (Linux Standard Base) to be installed as it uses the `/lib/lsb/init-functions` library.  

For example, if on a Red Hat system you get:
```
error reading information on service /etc/init.d/cdxd: No such file or directory
```

You will need to run:

```
sudo yum install redhat-lsb

```

### Initialization Script

As root, make a symbolic link to the `cdxd` script;  (will fail if the symlink exists already):

```
sudo ln -s /opt/cdx/bin/cdxd /etc/init.d/cdxd
```

To create or update the symlink:

```
sudo ln -sf /opt/cdx/bin/cdxd /etc/init.d/cdxd
```

### Link Initialization Script to Appropriate Run Levels

Execute the following command. This will add appropriate 'S' and 'K' entries in the given run-levels. This will also add appropriate sequence number by considering the dependencies.

```shell
sudo update-rc.d cdxd defaults
```

On RedHat you should use `chkconfig`:
```shell
sudo chkconfig cdxd on
```

By default, `chkconfig` assumes levels 2345. Any runlevels not specified as on, will be marked as off; levels 016 by default. You can specify runlevels thusly: `chkconfig --level 345 cdxd on`.

### Start the Service

Since this is not a reboot, you will have to start the service initially by hand:
```
sudo service cdxd start
```
Check the state of the service:

```
sudo systemctl status cdxd.service
sudo journalctl -xn
```

You can find the console logs in `/opt/cdx/cdxd.log`. This is invaluable for debugging your background service.

### Editing the Start Script

After editing the `cdxd` script, be sure to reload it:

```
sudo systemctl daemon-reload
```

To check if the service is running, execute this command:

```
service cdxd status
```

### Respawning

**Optional:** you can have the service respawn if it ever crashes or is forcefully terminated by adding a respawn line for this service at the bottom of the `/etc/inittab` file:

```
id:2345:respawn:/bin/sh /opt/cdx/bin/cdxd start
```

This may conflict with the service, so be careful when using `inittab` respawning.

### Removing the Service

Run the command below to disable the CDX Daemon Service:

```
sudo update-rc.d cdxd disable
```

Run the command below to remove all the appropriate 'S' and 'K' entries from the different run levels:

```
sudo update-rc.d cdxd remove
```

### Editing the Daemon CDX Service

The service runs the `/opt/cdx/cfg/daemon.json` job. Placing  `Job` configurations in the `daemon.json` configuration file allows them to run continually in the background. Just add a `Schedule` parameter to each `Job` configuration to control when that job runs.

The `Wedge` component can be replaced with your own components or once you have a `Job` configured. It is there only to keep the daemon running until you add your components. Without any jobs or components, the daemon will exit.

Note that it is listening to port 55289. It has no users defined so the API is disabled. It can only be reached by its monitoring endpoints`/metrics` and `/api/health`. Additionally, the IP Access Control List is rather restrictive. It only allows access from the local host and a few non-routable subnets. Adjust as needed.

When updating your CDX daemon job, your workflow might look something like this:

1. Edit the `daemon.json` file.
2. Restart the service with `sudo service cdxd restart`
3. Check the status of the service with `sudo systemctl status cdxd.service`
4. Check the service log in `cat /opt/cdx/cdxd.log` and look for errors with the service.
5. Check the daemon logs in `opt/cdx/log/cdxd*.log` a look for errors with the jobs.
6. If there are problems in the logs, go back to step 1.


## Windows

Running anything in the background on Windows is less than ideal, it is a windowing system after all. The closest one might come to running a process in the background is using the `START` command:

```
START /MIN cdx daemon.json
```

The program runs in its own window, but minimized.

You can also try:

```
START /B "" cdx daemon.json &
```

One problem with running services in this manner is that once the user exits Windows, all their processes exit as well.

Instead of using a `Service` to handle the scheduling of data transfer jobs, it might be better to run each `Job` in that service as its own `Job`, and using the Windows scheduler to handle the execution. This will not help when running web services or anything involving `HttpListener`.

The best approach is to run the job in a Java [wrapper](https://wrapper.tanukisoftware.com/doc/english/introduction.html) and run the wrapped job as a Windows Service. This is how many Java applications servers run.



# Managing Services

When services are run, they include a component that will listen for HTTP requests and respond to them. This component is called the `HttpManager` and allows services to to be monitored and managed across the network with a variety of tools.

The manager runs by default on port 55290, but can be configured to any available port:

```json
"Manager" : {
    "Port" : 80
}
```

### HTTPS - Secure Connections 

If you choose port 443, the server will be set into secure mode; requiring SSL handshaking to make the connection. It is possible to turn on SSL connections on any port by specifying the `SecureServer` configuration option:

```json
"Manager" : {
    "Port" : 80,
    "SecureServer": true
}
```
#### SSL Certificates

In order to enable SSL, you need certificates.  They should be installed in a key store named `keystore.jks` and placed in the classpath. The easiest way to do this is to place the `keystore.jks` file in the `cfg` sub-directory of the Coyote installation directory.

Here is an example of how to generate a self-signed certificate for the localhost:
```
 keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass password -validity 360 -keysize 2048 -ext SAN=DNS:localhost,IP:127.0.0.1  -validity 9999
```
This will generate a key store file named `keystore.jks` with a self signed certificate for a host named localhost with the IP address 127.0.0.1. Of course, you will probably want to use the real hostname and IP address of the host running CDX. If you are going to use this in production, you will probably want a certificate signed by a well-known certificate authority.

Again, it is important to place this key store file in the classpath. The CDX start-up scripts automatically place the configuration directory (`cfg`) of the application home in the classpath and this is the easiest location for your key store file.

For more protection, you might consider making the file read-only by the users running CDX. This way, only authorized accounts can manage the key store.

## Monitoring

It is possible to monitor the operation of the service through the `/api/health` endpoint on the monitor. It will respond with "UP" if the service is running. It is intentionally designed to be simple to help parsing the response and to keep processing at a minimum on the service.

Additionally, the `HttpManager` supports monitoring by [Prometheus](https://prometheus.io/) through the `/metrics` endpoint. Requests to this endpoint will return a set of operational metrics in the [OpenMetrics](https://openmetrics.io/) format. This means that [Prometheus](https://prometheus.io/) can use the both the `/api/health` endpoint for basic status monitoring and the `/meteics` endpoint for detailed operational metrics. When combined with tools like [Grafana](https://grafana.com/) CDX services can be completely monitored.

## Security

Because this feature is an attack vector, several security features have be designed into the `HttpManager` to help ensure the integrity of the system:

* **IP Access Control List** - By default, the manager only accepts connections from the local lost. Additional IP addresses (and subnets) can be added to the ACL.
* **Role Based Access Control** - access to functions within the manager can be controlled by user names and groups.
* **Denial of Service Detection** - The `HttpManager` can detect high volumes of requests and either throttle responses or blacklist the IP address.



### IP Access Control List

When a connection is received by the `HttpManager` the IP address of the client is checked to see if it is allowed to make a connection. By default only connection from the localhost are allowed. This can be configured:

```json
"Manager" : {
    "IPACL" : {
        "default" : "deny",
        "127.0.0.1" : "allow"
    }
}
```

The above is the default IP access control list configuration. To replace this configuration, add a `Manager` section to the configuration and specify your own ACL:

```json
"Manager" : {
    "IPACL" : {
        "default" : "deny", 
        "127.0.0.1" : "allow",
        "172.28.147.6/0" : "allow",
        "192.168/16" : "allow",
        "10/8" : "deny"
    }
}

```

The first line defines the default rule for access checks. If no other rule was matched, this rule will be used. 

The second line allows connections from the localhost. The third line allows access from a specific address. Note the `/0` at the end specifying the subnet mask to apply. The fourth line allows access to any host in the `192.168.0.0` subnet.

The fifth line disallows anything from the `10.0.0.0` subnet, although this is unnecessary since the default rule is to deny and any address that is not allowed by other rules will be denied by default.

The most insecure configuration is to allow all connections:
```
"Manager" : {
    "IPACL" : {
        "default" : "allow"
    }
}
```
### Authentication

All authentication is presented using Basic Authentication (i.e. "BasicAuth"). User names and passwords should be encoded in the request headers. These credentials will be checked against the users configured in the service.

```json
"Manager" : {
    "Auth": {
        "Users":[
            { 
                "Name" : "admin",
                "Password" : "secret",
                "Groups" : "sysop,devop"
            },
			{ 
			    "Name" : "bob", 
			    "Password" : "secret"
			}
		],
        "AllowUnsecuredConnections" : true,
        "SendAuthRequestOnFailure": true
	}
}
```

The above defines two users, `alice` and `bob`. Alice is a member of both the `devops` and `sysop` groups. Bob is not a member of any groups.

The `AllowUnsecuredConnections` configuration options is an additional security check. If set to `false` (the default) all authentication must occur across a secure, SSL connection.

The internal server can be configured to present authentication requests if no credentials are presented or if authentication fails by setting `SendAuthRequestOnFailure` to true. The user will then be asked to provide username and password credentials.

It is recommended that the usernames and password be encrypted using the internal encryption capabilities:
```json
"Manager" : {
	"Port": 443,
	"Auth": {
		"Users" : [ 
			{ "ENC:Name" : "cNcvHrTNJTyyUkgkTdSlDEeVLJMFuOYF", 
			  "ENC:Password" : "Gfkyh75cqGEXtR5jRgqlNzgAdPz/K2mz", 
			  "ENC:Groups" : "0XpjbievElMHAu12vnj/GyZwndbwd5zoUnVMDb1VaHU="
			},
			{ "ENC:Name" : "OdPhHiNBqMjlIAF1CqVJUg==",  
			  "ENC:Password" : "6yh75cRgqlNzgAdCIY8APrIlUPz/K2mz"
			}
		],
		"SendAuthRequestOnFailure": true,
		"AllowUnsecuredConnections" : false
	},
}
```

As mentioned in other labs, a `cipher.key` must be provided in an environment variable before the values can be decrypted. Otherwise, the default key is used and the values are not protected, only obfuscated.

## Command Interface

The `/api/cmd` endpoint allows a manager to issue commands to the service. 

All commands require the authentication and membership to either the `devop` or `sysop` group. This implies that the `Auth` configuration section be present in the service configuration and a user configured with membership to one of those roles.

### Shutdown

The most common of which is the shutdown command accessible through the `/api/cmd/shutdown` endpoint.