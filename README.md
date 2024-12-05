# 

# 

This infra automatically processes AWS Cost And Usage Reports. It prepares the
data stored on S3 in partitioned and formatted state via AWS Glue, making it
ready for querying using AWS Athena.

## How it works

<img src="https://i.imgur.com/8xwlWlp.png" alt="AWS Architecture" height="350" width="425">

1. AWS Generates CUR reports and stores them on S3. These reports contain
   detailed billing information for your account.
1. Changes in the CUR bucket trigger S3 event-notifications.
1. Glue Crawler is working in event-mode and monitors new messages in SQS coming
   from event-notifications. When new message arrives, the Crawler starts
   processing them almost in real-time. It only processes the updated or newly
   added objects. This approach is more efficient than the default crawling
   (going through each s3 object every tme). After that's done, the Glue data
   catalogue has the updated tables and schema that the Athena references in
   it's SQL queries.
1. If processing of messages by the Crawler fails 3x in a row, the message is
   sent to
   [SQS DLQ](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
1. Athena uses the contexts from Glue Catalogue to query the data from the CUR
   bucket

## Use cases

1. I think this setup could replace/offer better alternative to
   [Kubecost Cloud Costs](https://docs.kubecost.com/using-kubecost/navigating-the-kubecost-ui/cloud-costs-explorer)
   . For example, you can setup the Athena datasource in Grafana and create your
   custom dashboards with SQL. Since CUR consists of very detailed data, the
   possibilities for different views are large.