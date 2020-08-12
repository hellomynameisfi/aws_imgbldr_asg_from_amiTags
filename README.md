# EC2 Auto Scaling groups Instance Refresh (amiTag) solution

## Table of contents
* [General info](#general-info)
* [Setup](#setup)

## General info
These are step-by-step instructions on how to automatically update the ami-ID inside an Auto Scalig Group with an Image Builder generated image ami-ID. The goal is to have one function responsible for updating all desired ASG's instead of a function for each ASG separately (and because I was unsuccesfull with the original solution provided by aws-samples). This is done by reading a set amiTag value instead of a environment variable from within Lambda function.

This solution is building on "Sample EC2 Auto Scaling groups Instance Refresh solution" by aws-samples (https://github.com/aws-samples/ec2-auto-scaling-instance-refresh-sample).
	
## Setup
To run this project, install it locally using npm:

### Create a IAM Role for Image Builder
First of all we need to create a IAM role for our Image Builder pipeline. For that navigate to IAM in youe ARS Console (https://console.aws.amazon.com/iam/) and under Access management on the left panel select "Roles" (https://console.aws.amazon.com/iam/home?/roles#/) click "Create role" button.

After being taken to the next screen select: "AWS service" + "EC2" and click the "Next: Permissions" button at the bottom of the page.

Here you will need to add the following Policies: 
* AmazonSSMManagedInstanceCore 
* EC2InstanceProfileForImageBuilder 

Now click "Next: Tags" button on the bottom of the page (add Tags if needed by your setup) and again click "Next: Review" on the bottom of the page.

Here you will have to provide a name for your role. Let's call it: imagebuilder_pipeline_role. Click the "Create role" button at the bottom of the screen.

Search for the role you just created (https://console.aws.amazon.com/iam/home?/roles#/) and click on it. 

### Create your pipeline
asdasdasdasdsadasdasdd

### Create a IAM Role for Lambda
We also need to create the role for the Lambda function to be able to update the Auto Scaling Group. 

lambda_refresh_ami

### Create an SNS topic
asdasdasdasd

### Create a Lambda function
Now we need to create a Lambda function that will do all the job for us. To do that navigate to Lambda in your AWS Console (https://console.aws.amazon.com/lambda/) and click the "Create function" button. 
```
code
```
