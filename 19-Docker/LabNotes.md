# Containerized Coyote

Pull the image from [Docker Hub](https://hub.docker.com/r/coyotesys/coyote):
```
docker pull coyotesys/coyote
```

This approach assumes the use of volumes to mount directories:

* `cfg` holds the configurations to be called
* `wrk` holds the results of processing



## Custom Images

Once you have a working configuration, it is sometime easer to place it in a image so it can be called simply:

```
docker run --rm dataload
```

The above assumes you have an image named `dataload` the performs some processing. The `--rm` option removes the container when it terminates so you don't have loads of containers hanging around.





Building you own custom images is simply a matter of starting from

cd into the Java 8 directory

$ docker build -t java8 .



Assuming the image name of "coyote" 

Generate/encode an encrypted string
$ docker run coyote encrypt "my secret text"

Generate/encode an encrypted string using a specific key and encryption
$ docker run -e cipher.key='5up3rS3cret' -e cipher.name='xtea' coyote encrypt "my secret text"

Run a job configuration using a specific encryption key
$ docker run -e cipher.key='5up3rS3cret' coyote http://someplace.com/cfg/myjob.json