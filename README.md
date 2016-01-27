# rds-copy-snapshots-lambda
Makes a copy of the most recent auto snapshot and deletes ones older than a set number of months.

## Usage
This python script is a meant to be run as a scheduled AWS Lamdba function. You should have auto snapshots enabled on your RDS instances and this script will copy an auto snapshot so that it will not be deleted.  You will need to configure the following variables at the top of the script:

INSTANCES - List of database identifiers, or "all" for all databases
eg. ["db-name1", "db-name2"] or ["all"]

TAGS - Dictionary of tags to use to filter the snapshots. May specify multiple
eg. {'key': 'value'} or {'key1': 'value1', 'key2': 'value2', ...}

MONTHS - The number of months to keep ONE snapshot per month

REGION - AWS region in which the db instances exist
eg. "us-east-1"

## Configure Lambda function
### IAM Role Policy
Go to the IAM service in the AWS Management console. Click on Roles and click the Create New Role button. Name the role rds-copy-snapshots and click Next Step. Click the Select button that is next to the AWS Lambda service. On the Attach Policy page, don't check any boxes and just click Next Step. Click Create Role. Click on the newly created role and expand the Inline Policies and click where it says click here to create a new policy. Click Custom Policy and click Select. Name the policy rds-copy-snapshots. Copy the contents of the iam_role_policy.json file and paste it in the Policy Document box and click Apply Policy.

### Create Lambda function
#### Configure function
Go to the Lambda service in the AWS Management console. Create a new function and on the Select blueprint page click the Skip button. On the Configure function page fill in the following details:
* Name: rds-copy-snapshots
* Description: An AWS Lambda function that makes a copy of the most recent auto snapshot and deletes ones older than a set number of months
* Runtime: Python
* Code box: paste the contents of the rds-copy-snapshots-lambda.py file
* Handler: rds-copy-snapshots.main
* Role: rds-copy-snapshots
* Memory: 128
* Timeout: 10 sec
Click the Next button and click Create function.
In the Code tab, configure the variables at the top of the script to your desired configuration. Click Save.

#### Event sources
Click the Event sources tab and click the Add event source link. Choose the type Scheduled Event and fill in the following details:
* Name: rds-copy-snapshots
* Description: Run script monthly
* Schedule Expression: cron(30 11 1 * ? *)
Click submit and your function will run on the first day of every month at 11:30 UTC. You should change the time to be after your backup window configured on your db instances.

#### Test function
You can test the function from the Lambda console. Click the Actions button and select Configure test event. Choose Scheduled Event from the drop down. Add the following parameter to the structure "noop": "True".  This will tell the script to not actually delete any snapshots, but to print that it would have. Now you can press the Save and Test button and you will see the results of the script running in the Lambda console.

#### CloudWatch logs
You will be able to see the output when the script runs in the CloudWatch logs. Go to the CloudWatch service in the AWS Management console. Click on Logs and you will see the rds-copy-snapshots log group. Click in it and you will see a Log Stream for every time the script is executed which contains all the output of the script. Go back to the Log Group and click the Never Expire link in the Expire Events After column of the log group row. Change the Retention period to what you feel comfortable with.