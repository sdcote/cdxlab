# AlpineLinux with a glibc-2.23, Oracle Java 8, and coyote web server
FROM alpine:java
MAINTAINER Steve Cote <sdcote@gmail.com> 

# Add the files from the build context
ADD opt opt

# Expose the HTTP server port
EXPOSE 80

# Run the server
ENTRYPOINT ["java -Dapp.home=/opt/coyotedx -jar /opt/coyotedx/lib/CoyoteWeb-0.0.3.jar /opt/coyotedx/cfg/app.json"]

# EOF