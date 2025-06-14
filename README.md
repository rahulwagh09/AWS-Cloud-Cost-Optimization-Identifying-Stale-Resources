# AWS Cloud Cost Optimization - Identifying Stale Resources
Identifying Stale EBS Snapshots
In this example, we'll create a Lambda function that identifies EBS snapshots that are no longer associated with any active EC2 instance and deletes them to save on storage costs.

## Description:
The Lambda function fetches all EBS snapshots owned by the same account ('self') and also retrieves a list of active EC2 instances (running and stopped). For each snapshot, it checks if the associated volume (if exists) is not associated with any active instance. If it finds a stale snapshot, it deletes it, effectively optimizing storage costs.

step1: Create a ec2 instance. When we a create a ec2 instance a volume is attached to it.

![image](https://github.com/rahulwagh09/Projects/assets/128569400/2541d80f-a399-4e00-a2c0-80d82816671e)

volume associated with the My web server ec2 instance:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/43bc2250-684f-40c7-8d31-d28be052ef62)

Now we will create snapshot of ebs volume of this instance:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/fb3908d8-1e98-4e12-ba22-1521671e9f6f)

Let's suppose in the future this instance is no longer needed and a person deleted the instance and volumes attached to it, but he has taken many snapshots and forgot to delete those snapshots, so we will write a lambda function that will check and delete those unused snapshots.

Currently, we have only one snapshot of the volume and that volume is attached to the instance right now so it will not delete the snapshot:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/5e178a4b-a2c0-44a3-9280-fb81d52f7d9c)

Use the below python code:
("ebs_stale_snapshosts.py" provide in this file)

![image](https://github.com/rahulwagh09/Projects/assets/128569400/b4e7cf91-58d5-423f-84ee-1195133b2b33)

First click on deploy button to save the code then click on test button and create test event:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/ebb5fc56-3e4f-425a-b33b-8ad3834547b5)

Increase the default execution time of our function from 3 to 10 seconds as it is a bit bigger code so 3 seconds are not enough:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/4c020109-f9ca-4ace-b9df-30edb263349c)

But this function has not the permission to describe the ebs volumes so that we will assign permission:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/6c1afe15-434c-4a50-8a69-9b255c418c07)

We will create new policy for this role so that it can describe ebs volumes:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/36f7f32d-61a3-4b65-b8c0-3a5b3bc0770d)

Create a New policy and attach the necessary permissions:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/1d04fadd-b447-4d2a-9852-b5a679a21db2)

Now our role has this new policy also:

![image](https://github.com/rahulwagh09/Projects/assets/128569400/8af5f817-4727-4ea9-bdeb-c005a6d8e501)

Now try to execute the function, but it should not delete the snapshot as snapshot of volume is currently attached to ec2 instance:

The error response stats it needs additional permission. Add the Describe volumes and DescribeInstances to the policy created.

![image](https://github.com/rahulwagh09/Projects/assets/128569400/e2202e32-f765-4395-b4c3-334ee163634d)

Testing the function after the required permission are added to the policies.

![image](https://github.com/rahulwagh09/Projects/assets/128569400/342f6f8c-2efe-4e76-95d5-d282cd5b5509)

Now delete the instance it will delete the respective volume. Let us run the lambda function again it will delete the snapshot as the snapshot is attached to any existing instances.

![image](https://github.com/rahulwagh09/Projects/assets/128569400/5949be4e-c95a-4e9b-b5f2-ff868d263dd3)

### The scope of this project is very big, you can use this idea for any aws service like S3 buckets, EKS etc.
### Thats all in this project.
## Thank You 





=====================================================================

## AWS Cloud Cost Optimization - Identifying Stale EBS Snapshots
### Overview
This AWS Lambda function is designed to manage Amazon Elastic Block Store (EBS) snapshots. It automatically deletes snapshots that are not attached to any active EC2 instance or whose associated volumes have been deleted. Additionally, snapshots older than 30 days that are not associated with active volumes are also deleted.

### Prerequisites
AWS Account: You need an AWS account to deploy and run the Lambda function.

IAM Role with Permissions: The Lambda function requires an IAM role with the following permissions:
```
ec2:DescribeSnapshots
ec2:DescribeInstances
ec2:DescribeVolumes
ec2:DeleteSnapshot
```
Ensure the IAM role associated with the Lambda function has these permissions.

### AWS Lambda: You need an active Lambda function setup. This Lambda function can be invoked on a schedule using AWS CloudWatch Events to run periodically (e.g., daily).

### Steps to Set Up
Create the Lambda Function:

Go to the AWS Lambda Console.
Click Create function.
Choose Author from scratch and provide the necessary details such as function name and runtime (Python 3.x).
Add Required IAM Permissions:

Attach an IAM role with the following permissions to the Lambda function:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeSnapshots",
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DeleteSnapshot"
      ],
      "Resource": "*"
    }
  ]
}
```
### Deploy the Code:

Copy the Lambda function code (provided above) and paste it into the Function code section of the AWS Lambda console.
*** Click Deploy to save the changes.
### Set Up CloudWatch Event for Scheduling:


* Navigate to the CloudWatch Console.


* In the Events section, create a new rule to trigger the Lambda function on a schedule (e.g., every day).


* Define the rate or cron expression for the schedule (e.g., rate(1 day)).


* Modifying for 3-Minute Inactivity so that we can our lambda function quickly

#### To configure the Lambda function to delete snapshots that are not attached to any EC2 instance or whose associated volumes have been deleted after 3 minutes, follow these steps:

### Update the Time Check in the Lambda Function Code:


* Modify the part of the code that checks how long a snapshot has been inactive. Instead of deleting snapshots older than 30 days, we will delete them after 3 minutes.
### Code Changes:

Replace the line checking for a snapshot older than 30 days with a check for a snapshot that is older than 3 minutes. Update the relevant section of your code as follows:
```
# Replace this section:
if datetime.utcnow() - snapshot_start_time > timedelta(days=30):

# With this section:
if datetime.utcnow() - snapshot_start_time > timedelta(minutes=3):
```

### Test the Lambda Function:
Test the function using sample data in the Test section of the Lambda console.
Ensure that the function executes without errors and handles snapshots and volumes correctly.


## Notes
The Lambda function checks for snapshots that are not attached to any volume and deletes them. If the snapshot is associated with a volume that has been deleted, the snapshot is also deleted.
Snapshots older than 3 minutes from unattached volumes are automatically deleted.
Acknowledgments

I would like to express my gratitude to my teacher, Abhishek Veeramalla, for his invaluable guidance and support throughout the development of this project. His expertise and dedication have been instrumental in shaping my understanding and skills. I highly recommend checking out his insightful tutorials on his YouTube channel Abhishek Veeramalla for further learning and inspiration.
