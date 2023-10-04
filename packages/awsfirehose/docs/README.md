# Amazon Kinesis Data Firehose
Amazon Kinesis Data Firehose integration offers users a way to stream logs from Firehose to Elastic Cloud.
This integration includes predefined rules that automatically route AWS service logs to the respective integrations, which
include field mappings, ingest pipelines, predefined dashboards and ect. Here is a list of log types that are supported
by this integration:

| AWS service log    | Log destination           |
|--------------------|---------------------------|
| API Gateway        | CloudWatch                |
| CloudTrail         | CloudWatch                |
| Network Firewall   | Firehose, CloudWatch, S3  |
| Route53 Public DNS | CloudWatch                |
| Route53 Resolver   | Firehose, CloudWatch, S3  |
| VPC Flow           | Firehose, CloudWatch, S3  |
| WAF                | Firehose, CloudWatch      |

## Limitation
It is not possible to configure a delivery stream to send data to Elastic Cloud via PrivateLink (VPC endpoint). 
This is a current limitation in Firehose, which we are working with AWS to resolve.

## Instructions
1. Install the relevant integrations in Kibana

    In order to make the most of your data, install AWS integrations to load index templates, ingest pipelines, and 
    dashboards into Kibana. In Kibana, navigate to **Management** > **Integrations** in the sidebar.
    Find the **AWS** integration by searching or browsing the catalog.
    
    ![AWS integration](../img/aws.png)
    
    Navigate to the **Settings** tab and click **Install AWS assets**. Confirm by clicking **Install AWS** in the popup.
    
    ![Install AWS assets](../img/install-assets.png)

2. Create a delivery stream in Amazon Kinesis Data Firehose

    Sign into the AWS console and navigate to Amazon Kinesis. Click **Create delivery stream**.
    Configure the delivery stream using the following settings:
    
    ![Amazon Kinesis Data Firehose](../img/aws-firehose.png)
    
    **Choose source and destination**
    
    Unless you are streaming data from Kinesis Data Streams, set source to Direct PUT (see Setup guide for more details on data sources).
    
    Set destination to **Elastic**.
    
    **Delivery stream name**
    
    Provide a meaningful name that will allow you to identify this delivery stream later.
    
    ![Choose Firehose Source and Destination](../img/source-destination.png)
    
    **Destination settings**

    1. Set **Elastic endpoint URL** to point to your Elasticsearch cluster running in Elastic Cloud.
    This endpoint can be found in the Elastic Cloud console. An example is https://my-deployment-28u274.es.eu-west-1.aws.found.io.

    2. **API key** should be a Base64 encoded Elastic API key, which can be created in Kibana by following the
    instructions under API Keys. If you are using an API key with “Restrict privileges”, be sure to review the Indices
    privileges to provide at least "auto_configure" & "write" permissions for the indices you will be using with this
    delivery stream.

    3. We recommend leaving **Content encoding** set to **GZIP** for improved network efficiency.

    4. **Retry duration** determines how long Firehose continues retrying the request in the event of an error.
    A duration of 60-300s should be suitable for most use cases.

    5. Elastic requires a **Buffer size** of `1MiB` to avoid exceeding the Elasticsearch `http.max_content_length`
    setting (typically 100MB) when the buffer is uncompressed.

    6. The default **Buffer interval** of `60s` is recommended to ensure data freshness in Elastic.

    7. **Parameters**

       1. Elastic recommends setting the `es_datastream_name` parameter to `logs-awsfirehose-default` in order to
       leverage the routing rules defined in this integration. If this parameter is not specified, data is sent to the
       `logs-generic-default` data stream by default.
       ![Firehose Destination Settings](../img/destination-settings.png)

       2. The **include_cw_extracted_fields** parameter is optional and can be set when using a CloudWatch logs subscription
       filter as the Firehose data source. When set to true, extracted fields generated by the filter pattern in the
       subscription filter will be collected. Setting this parameter can add many fields into each record and may significantly
       increase data volume in Elasticsearch. As such, use of this parameter should be carefully considered and used only when
       the extracted fields are required for specific filtering and/or aggregation.

       3. The **include_event_original** field is optional and should only be used for debugging purposes. When set to `true`, each
       log record will contain an additional field named `event.original`, which contains the raw (unprocessed) log message.
       This parameter will increase the data volume in Elasticsearch and should be used with care.

3. Send data to the Firehose delivery stream

    Consult the [AWS documentation](https://docs.aws.amazon.com/firehose/latest/dev/basic-write.html) for details on how to
    configure a variety of log sources to send data to Firehose delivery streams.