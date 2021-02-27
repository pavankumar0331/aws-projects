# ecr-image-cross-region-replication

# Objective
The main objective or intent for building the solution is to copy the ecr images from one aws region to another aws region.

# Why to use this solution??
    1. To make images highly available.
    2. Highly redundent.

# Background
This end to end solution leverages the below listed aws services. <br />
    1. AWS CodeBuild <br />
    2. ECR <br />
    3. SQS <br />
    4. Lambda <br />
    5. S3 <br />
    6. Cloudwatch event. <br />

# How this solution works??
To begin with, you should decide which ecr repository images that you would want to copy from one region to another region. 
A cloud watch event will be created and SQS ( FIFO ) will be added as target to the event. 
Whenever an image is successfully pushed to ecr repository cloudwatch event sent that notification to SQS Queue. 
Once the notification is stored in SQS, it then sent to consumer (i.e AWS Lambda).
AWS Lambda will extract a few details from the Message like ( ECR Repository name, ImageName ) and start the codebuild and delete the message from the Queue.
AWS Codebuild then do the following stuff. <br />
    a. clone the image from source ecr repo. <br />
    b. create the ecr repo in destination region ( if not exist ). <br />
    c. push the image to destination region. <br />

# Pre-requisities
An existing ECR Repository

## How to deploy??
Note: We will be using AWS Cloudformation to deploy this solution. If u are deploying as user, ask the AWS account administrator to grant the above listed
AWS services permissions.

Follow along to deploy this solution.

Step1: Run the following command to create an s3 bucket and to copy the buildspec file to input folder.
Note: <br />
  Give name to bucket by replacing <BUCKETNAME> <br />
  set region value by replacing <REGION> <br />
  
aws s3 mb s3://<BUCKETNAME>/ <br />
aws s3 --region <REGION> cp buildspec.yml s3://cross-region-ecr-replication-project-input-store/input/buildspec.yml <br />

Step2: Run the following command to deploy cloudformation stack. <br />
Note: <br />
    Set region value by replacing <REGION>. Make sure deploy s3 bucket and CF stack in the same region. <br />
    Set stackname by replacing <StackName>. <br />
  Replace <ECRRepoName> with ecr reponame that you want to replicate. <br />
  Replace <Destregion> with aws ecr region. this sould be the region where your ecr images would be replicated to. <br />
  Replace <BUCKETNAME> with the name that you gave in step1. <br />
  
aws --region <REGION> cloudformation create-stack --stack-name <StackName> --template-body file://cross-region-image-copy-template.yaml --capabilities CAPABILITY_NAMED_IAM --parameters '[{"ParameterKey": "ECRRepoName", "ParameterValue": "<ECRRepoName>"}, {"ParameterKey": "Destregion", "ParameterValue": "<Destregion>"}, {"ParameterKey": "BucketName", "ParameterValue": "<BUCKETNAME>"}]'











 
