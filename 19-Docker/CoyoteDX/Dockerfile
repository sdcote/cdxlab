# AlpineLinux with a glibc-2.23, Oracle Java 8, and Coyote DX daemon
FROM alpine:java
MAINTAINER Steve Cote <sdcote@gmail.com> 

ENV PATH=${PATH}:/opt/coyotedx/bin

# Add the files from the build context
ADD opt opt

# Expose the web manager port
EXPOSE 55290

# Run the Coyote DX
ENTRYPOINT ["cdx"]
# ...as a daemon by default
CMD ["daemon.json"]
     
# EOF