# Process

This lab covers how to incrementally develop a data transfer job.

While it is possible to copy and paste a job and modify its values, there are times when data discovery is the only way to efficiently create a job.

## Extract-Transform-Load (ETL)

In this example, we will be retrieving data from a web service and placing it in a relational database.

1. Call the web service and examine its output (`ETL1.json`).
2. Call the web service and profile the data (`ETL2.json`).



## Web Services

When setting up an endpoint to receive REST calls, such as listening for webhooks, it is best to adopt an incremental approach:

1. Listen for data and log everything (`WS1.json`).
2. Analyze the data and determine a processing approach.
3. Implement your configuration while logging everything (`WS2.json`).
4. Streamline your listener (`WS3.json`).
