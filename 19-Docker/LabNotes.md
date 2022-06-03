# Containerized CDX

Pull the image from [Docker Hub](https://hub.docker.com/r/coyotesys/coyote):
```
docker pull coyotesys/cdx
```





Assuming the image name of "cdx" 

Generate/encode an encrypted string

```shell
$ docker run cdx encrypt "my secret text"
```

Generate/encode an encrypted string using a specific key and encryption

```shell
$ docker run -e cipher.key='5up3rS3cret' -e cipher.name='xtea' cdx encrypt "my secret text"
```

Run a job configuration using a specific encryption key

```shell
$ docker run -e cipher.key='5up3rS3cret' cdx http://someplace.com/cfg/myjob.json
```


## Base Image

The base image contains the core project and several of the more popular modules. You can check this with the following command:
```
$ docker run --rm cdx --version
Loader   0.7.22
CoyoteWS 0.2.0-dev
CoyoteMT 0.1.1-exp
CoyoteMC 0.0.2-dev
CoyoteMQ 0.1.1-dev
Coyote   0.8.7-dev
CoyoteFT 0.0.1-dev
CoyoteDB 0.1.23-dev

$
```
In the above you can see Coyote 0.8.7 is the base and there are several other modules of differing versions.

### User Directory

At the time of writing, the current working directory for all CDX images is the root directory.

```
'user.dir' = /
```



## Batch Runs
In this scenario, CDX data transfer jobs are run in an ad hoc fashion where the job runs and the container terminates after the last record in the batch is processed. 

The standard way to pass a configuration file is to use volumes and place the file in the configuration directory:

```shell
$ docker run -v /local/path/to/file.json:/opt/cdx/cfg/file.json cdx file.json
```

Sometimes it's easier to just reference the local file. Here we are using the shell command `pwd` to print out the current working directory. You can use `$HOME` or other variable substitution supported by your shell. Windows is, of course, different and you can substitute the appropriate variables as desired.

```shell
$ docker run -v `pwd`/file.json:/opt/cdx/cfg/file.json cdx file.json
```

File extensions are not strictly necessary:

```shell
$ docker run -v `pwd`/file:/opt/cdx/cfg/file cdx file1
```

Of course, the source filename is not required to be the same as the configuration name:

```shell
$ docker run -v `pwd`/myconfig.json:/opt/cdx/cfg/server cdx server
```

If you want to save the results of processing, volumes are, again, the answer. Map the working directory of the container to the local file system:

```shell
$ docker run -v `pwd`/:/opt/cdx/wrk -v ./file.json:/opt/cdx/cfg/file.json cdx file.json
```

The most complete version of the call would be to remove the container after it exits:

```shell
$ docker run --rm -v `pwd`/:/opt/cdx/wrk -v ./file.json:/opt/cdx/cfg/file.json cdx file.json
```

## Server Runs
The CDX data transfer job can remain running in a container as a service on the host. This is dependent on the configuration file used. Volumes are used to specify the configuration file as described above.

The only difference is that the container may need to be restarted when the server is rebooted.

```shell
$ docker run --restart unless-stopped -v ./:/opt/cdx/wrk -v ./server.json:/opt/cdx/cfg/server cdx server
```

Of course, you may want to run the container detached from the console as a named container:

```shell
$ docker run -d --name webservice --restart unless-stopped -v ./:/opt/cdx/wrk -v ./server.json:/opt/cdx/cfg/server cdx server
```

Then use `docker logs` to see the output of the named container:

```shell
$ docker logs webservice
```

If the service listens to a port, it may need to be exposed on the host platform. This is accomplished with port mapping. If your service listens to port `8080` and you want it to be exposed as port `80` on the host platform use the `-p` mapping option:

```shell
$ docker run -d --name webservice -p 8080:80 --restart unless-stopped -v ./:/opt/cdx/wrk -v ./server.json:/opt/cdx/cfg/server cdx server
```

# Lab 1 - Examples

The following are a few example sessions to illustrate how CDX can be run in docker to perform various tasks.

## Test Run

Make sure your are in the `lab1` directory for this example.

In this example we will use the `TestRun.json` in the current directory, map the current directory to the `cfg` directory in the container, and call CDX to run it:
```shell
$ docker run --rm -v `pwd`:/opt/cdx/cfg cdx TestRun.json
```
If you have multiple jobs defined in your current directory, you can call all of them, but by changing the last argument to specify the job to run. This is so useful, that some users create an alias for it.

## Simple Run
Make sure you are in the `lab1` directory for this example.

In previous labs, we use a simple job to read a file in one format and write it out to another. We will use that same example here, only with the CDX docker image.

If we run he following command like the above, we don't seem to get anything:
```shell
$ docker run --rm -v `pwd`:/opt/cdx/cfg cdx Simple.json
```
The reason for this is all the processing occurred in the container, including the output of the new file. Nothing is written out on the host.

We can verify this by passing the `-d` argument to CDX to enable debug logging:

```shell
$ docker run --rm -v `pwd`:/opt/cdx/cfg cdx -d Simple.json
```
You should see that the `FlatFileWriter` is using a target of `/opt/cdx/wrk/Simple/users.txt` which lives in the container. Since we use the Docker `--rm` argument, Docker removes the container when it terminates and the file is lost.

The way around this is to also map the work directory of the container to a point in the local file system so all output will be written on the host. The following maps the current working directory to both the `/opt/cdx/cfg` and  `/opt/cdx/wrk` in the container so CDX will read its configuration from and write its output to the current working directory.

```shell
$ docker run --rm -v `pwd`:/opt/cdx/cfg -v `pwd`:/opt/cdx/wrk cdx Simple.json
```
Now you will see a `Simple` directory in your current working directory, and within that directory is our results:

```shell
$ ls -las ./Simple/
total 4
0 drwxr-xr-x 2 root    root    22 Jun  2 12:16 .
0 drwxr-xr-x 3 sdcote  users   72 Jun  2 12:16 ..
4 -rw-r--r-- 1 root    root  1900 Jun  2 12:16 users.txt
$
```
As mentioned before, this command is so useful that many users create an alias for it.


# Custom Images

Once you have a working configuration, it is sometimes easier to place it in an image so it can be called simply:

```shell
docker run --rm dataload
```

The above assumes you have an image named `dataload` that performs some processing. The `--rm` option removes the container when it terminates so you don't have loads of containers hanging around.



Make sure you are in the `lab2` directory.

Open the `Dockerfile` and view its contents:

```dockerfile
# Start form the base CDX image
FROM cdx

# Add the files from the build context
ADD opt opt

# Expose the HTTP server port
EXPOSE 80

# Run the server
CMD ["webhookproxy"]

# EOF
```

In the above example, you see the first lines start the Docker build from the `cdx` image that you have previously downloaded. 

The second few lines updates the `/opt` directory with the configuration files from the local file system. This could easily contribute more to the `lib` directory as well.

The last few lines expose port 80 and add the `webhookproxy` command-line argument to the existing `ENTRYPOINT` specified in the base `cdx` image so when the container is created, CDX is called with the `webhookproxy` argument that will cause that configuration file to be loaded.

In this directory, you can build your custom image with:

```shell
$ docker build -t webhookproxy .
```

You then should have a docker image called `webhookproxy` that can be called easily:

```shell
docker run webhookproxy
```

The `webhookproxy` image should now be running in the background listing to requests on port 80 and sending them on to whatever URL is configured in the configuration file.



