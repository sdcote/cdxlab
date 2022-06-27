FROM cdx

# Create a user and a group (change UID and GID if they conflict on your system)
ARG UNAME=testuser
ARG UID=1000
ARG GID=1000
RUN groupadd -g $GID -o $UNAME
RUN useradd -m -u $UID -g $GID -o -s /bin/sh $UNAME
USER $UNAME

# Change our work directory to enable simple volume mapping
WORKDIR /wrk

# Run CDX overriding the work directory to our current directory set above
ENTRYPOINT ["cdx -owd"]

# Run the daemon configuration by default
CMD ["daemon.json"]
     
# EOF