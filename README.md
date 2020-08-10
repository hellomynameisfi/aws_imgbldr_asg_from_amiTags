# aws_imgbldr_asg_from_amiTags
Link to base material: https://github.com/aws-samples/ec2-auto-scaling-instance-refresh-sample

## Table of contents
* [General info](#general-info)
* [Technologies](#technologies)
* [Setup](#setup)

## General info
This project is simple Lorem ipsum dolor generator.
	
## Technologies
Project is created with:
* Lorem version: 12.3
* Ipsum version: 2.33
* Ament library version: 999
	
## Setup
To run this project, install it locally using npm:

```
import boto3
from botocore.exceptions import ClientError
import json
import logging
import os

logger = logging.getLogger()
logger.setLevel(logging.INFO)

# define boto3 clients
ec2_client = boto3.client('ec2')
asg_client = boto3.client('autoscaling')

def get_ami_id_from_ib_notification(ib_notification):
	# Parse Image Builder notification and look up AMI for Lambda region
	for resource in ib_notification['outputResources']['amis']:
		if resource['region'] == os.environ['AWS_REGION']:
			return(resource['image'])
		else:
			return(None)

def get_asg_name_from_ib_notification(ib_notification):
	return(ib_notification['distributionConfiguration']['distributions'][0]['amiDistributionConfiguration']['amiTags']['name_of_your_amiTag_key_from_pipeline'])

def get_launch_template_id_for_asg(asg_name):
	try:
		asg_describe = asg_client.describe_auto_scaling_groups(
			AutoScalingGroupNames=[asg_name])

		# Make sure the Auto Scaling group exists
		if len(asg_describe['AutoScalingGroups']) == 0:
			raise ValueError(
				 "The configured Auto Scaling group "
				 "does not exist: {}".format(asg_name))

		asg_details = asg_describe['AutoScalingGroups'][0]

		# ASG may have a LaunchTemplate, MixedInstancePolicy or LaunchConfiguration
		if 'LaunchTemplate' in asg_details.keys():
			return(asg_details['LaunchTemplate']['LaunchTemplateId'])
		elif 'MixedInstancesPolicy' in asg_details.keys():
			return(asg_details['MixedInstancesPolicy']
							  ['LaunchTemplate']
							  ['LaunchTemplateSpecification']
							  ['LaunchTemplateId'])
		else:
			return(None)

	except ClientError as e:
		logging.error("Error describing Auto Scaling group.")
		raise e


def create_launch_template_version_with_new_ami(lt_id, ami_id):
	try:
		latest_lt_version = ec2_client.describe_launch_templates(
			LaunchTemplateIds=[lt_id])['LaunchTemplates'][0]['LatestVersionNumber']

		response = ec2_client.create_launch_template_version(
						  LaunchTemplateId=lt_id,
						  SourceVersion=str(latest_lt_version),
						  LaunchTemplateData={'ImageId': ami_id})
		logging.info("Created new launch template version for {} : {} with "
					 "image {}".format(lt_id, str(latest_lt_version), ami_id))
		return(response)

	except ClientError as e:
		logging.error('Error creating the new launch template version')
		raise e


def trigger_auto_scaling_instance_refresh(asg_name, strategy="Rolling",
										  min_healthy_percentage=90, instance_warmup=300):

	try:
		response = asg_client.start_instance_refresh(
			AutoScalingGroupName=asg_name,
			Strategy=strategy,
			Preferences={
				'MinHealthyPercentage': min_healthy_percentage,
				'InstanceWarmup': instance_warmup
			})
		logging.info("Triggered Instance Refresh {} for Auto Scaling "
					 "group {}".format(response['InstanceRefreshId'], asg_name))

	except ClientError as e:
		logging.error("Unable to trigger Instance Refresh for "
					  "Auto Scaling group {}".format(asg_name))
		raise e


def lambda_handler(event, context):

	# Load SNS message body
	ib_notification = json.loads(event['Records'][0]['Sns']['Message'])

	asg_name = get_asg_name_from_ib_notification(ib_notification)

	# Finish if Image build wasn't successful
	if ib_notification['state']['status'] != "AVAILABLE":
		logging.warning("No action taken. EC2 Image build failed.")
		return("No action taken. EC2 Image build failed.")

	# Get the AMI ID for current region from Image Builder notification
	ami_id = get_ami_id_from_ib_notification(ib_notification)
	if ami_id is None:
		logging.warning("There's no image created for region {}".format(
						os.environ['AWS_REGION']))
		return("No AMI id created for region {}".format(
				os.environ['AWS_REGION']))

	# Get Launch Template id for the Auto Scaling group
	lt_id = get_launch_template_id_for_asg(asg_name)
	if lt_id is None:
		raise ValueError("Auto Scaling group {} doesn't use a "
						 "Launch Template".format(asg_name))

	# Get latest version for Launch template to use as source and create
	# new version with new AMI
	create_launch_template_version_with_new_ami(lt_id, ami_id)

	# Trigger Auto Scaling group Instance Refresh
	trigger_auto_scaling_instance_refresh(asg_name)

	return("Success")
```
