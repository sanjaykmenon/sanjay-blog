---
draft: False
date: 2024-03-28
categories:
  - AWS, data engineering
---


# Cloudformation 

I've been trying to setup a purely AWS based Data Engineering curriculum that folks can use to learn AWS in the DE space as I did not find anything myself.

For one the first projects, I planned to use the NYC Taxi Cab data set in CSV format, drop them in an S3 bucket, use a Lambda function to convert them to parquet and store them in a separate bucket.

The following resources are required.
 - Incoming S3 Bucket
 - Output S3 Bucket (that will contain the parquet files)
 - an SQS queue 
 - Lambda Function

 The SQS acts as a landing zone for all S3 Event triggers that gets created everytime an object is added to the incoming S3 bucket. Why is there a need for an SQS you may ask? We can always have S3 Event triggers act as direct triggers for a Lambda function to execute. Well, in the event of a failed S3 event (yes it is an event pun!), we either have to write logic to ensure that the Lambda function adds all the information of the failed S3 event and write it into a queue anyway that can be processed later. We are solving this by adding an SQS layer instead.

 Visually, what we are trying to do is, instead of

Option 1:  CSV File added to S3 bucket --> S3 Event Created --> Lambda Trigger to process CSV file

we will architect

Option 2:  CSV File added to S3 Bucket --> S3 Event Created --> SQS --> Lambda Trigger to process CSV file.

Option 2 is a better architected solution than Option 1. As to why, i leave that as an exercise to the reader.

An issue i've faced is that for now I'd used one cloudformation template for all of these infrastructure. Because an s3 event is resource property of a Bucket resource, and it needs to be pointed to an SQS we have a problem. Since the SQS itself is being created in the CFN template, Cloudformation is not able to recognize the ARN of the SQS (understandably). This isn't good design. A possible option is to have the SQS created separately in another CFT first, and then deploy the stack for the buckets so that CFN can get the ARN for the SQS.

The other option would be exploring the Stack Sets approach. I'm trying to ensure that I have a uniform deployment process for setting up this infrastructure from a production standpoint. If I were to have to separate CFTs for the SQS and the S3 Bucket, then my deployment pipeline needs to be setup in a way that the order of resource creation is always maintained. This is tech debt? Using StackSets means i'd define the CFTs and have them stored in S3 and then just define the Stack in a single CFT YAML. More to look into tomorrow, if I'm not doing LLM stuff.

