# Getting-started-with-AWS-Lambda
Deploy and configure an AWS Lambda-based serverless computing solution

# Overview
Create 2 lambda functions to retrieve a set of records created in a MySQL database present in an EC2 instance. These Lambda functions work with SNS and notifications are sent to subscribers of the SNS Topic.
<div align=center>
  <img src="https://github.com/cooksies/Getting-started-with-AWS-Lambda/assets/75002188/780f50b6-f5e3-4dde-9969-739056b3abf1">
</div>

Step | Details
--- | ---
1 | An Amazon CloudWatch Events event calls the salesAnalysisReport Lambda function at 8 PM every day Monday through Saturday.
2 | The salesAnalysisReport Lambda function invokes another Lambda function, salesAnalysisReportDataExtractor, to retrieve the report data.
3 | The salesAnalysisReportDataExtractor function runs an analytical query against the café database (cafe_db).
4 | The query result is returned to the salesAnalysisReport function.
5 | The salesAnalysisReport function formats the report into a message and publishes it to the salesAnalysisReportTopic Amazon Simple Notification Service (Amazon SNS) topic.
6 | The salesAnalysisReportTopic SNS topic sends the message by email to the administrator.

# Observing the IAM role settings
## Observing the salesAnalysisReport IAM role settings
  1. In the AWS Management Console go to the **IAM Dashboard** and click and in the navigation pane choose **Roles**
  2. In the roles search box, enter `sales`
  3. From the filtered results, click on the **salesAnalysisReportRole** hyperlink
  4. Choose the **Trusted Permissions**. Notice that lambda.amazonaws.com is listed as trusted. Lambda service can use this role
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
  5. Go Back to the **Permissions** tab. There are 4 policies assigned to the role. Click the + icon to see the policies granted
### Observing the salesAnalysisReportDERole IAM role settings
  6. In the navigation pane, click on **Roles** and search `sales` again
  7. Now click on the **salesAnalysisReportDERole** hyperlink
  8. Choose the **Trusted Permissions**. Same as the salesAnalysisReport, lambda.amazonaws.com is listed as trusted. Lambda service can use this role.
  9.  Go Back to the **Permissions** tab. There are 2 policies assigned to the role. Click the + icon to see the policies granted

# Creating a Lambda layer and a data extractor Lambda function
  10. Download these lab files to your local machines.\
      [pymysql-v3.zip](https://github.com/cooksies/Getting-started-with-AWS-Lambda/blob/main/pymysql-v3.zip)\
      [salesAnalysisReportDataExtractor-v3.zip](https://github.com/cooksies/Getting-started-with-AWS-Lambda/blob/main/salesAnalysisReportDataExtractor-v3.zip)

## Creating a Lambda layer
  11. In the AWS Management Console, search and go to **Lambda**
  12. In the navigation pane, choose **Layers** and then **Create layer**
  13. Configure the following layer settings:

Action | Input
--- | ---
Name | `pymysqlLibrary`
Description | `PyMySQL library modules`
Radio button |  Upplaod a .zip file
Upload button |  upload the [pymysql-v3.zip](https://github.com/cooksies/Getting-started-with-AWS-Lambda/blob/main/pymysql-v3.zip) file
Compatible runtimes | Python 3.9

  14. Click on **Create**
    - The message should display "Successfully created layer pymysqlLibrary version 1"

## Creating a data extractor Lambda function
  15. In the navigation pane, choose **Functions**
  16. Click on **Choose function**

Action | Input
--- | ---
Radio options | Author from scratch
Function Name | salesAnalysisReportDataExtractor
Runtime | Pythin 3.9
Change default execution role | 
Execution role | Use an existing role
Existing role | salesAnalysisReportDERole

  17. Create function

## Adding the Lambda layer to the function
  17. In the Function overview panel, scroll down to **Layers**
  18. Choose **Add a layer** and configure the following:
      
Action | Input
--- | --- 
Choose a layer | Custom Layers
Customer Layers |pymysqlLibrary
Version | 1

  19. Choose **Add**
## Importing the code for the data extractor Lambda function
  20. Scroll down to **Runtime settings**
  21. Choose **Edit** and configure the following:

Action | Input
--- | ---
Handler | `salesAnalysisReportDataExtractor.lambda_handler`

  22. Under **Code source**, choose **upload from** and choose **.zip file**
  23. Click on Upload and select the [salesAnalysisReportDataExtractor-v3.zip](https://github.com/cooksies/Getting-started-with-AWS-Lambda/blob/main/salesAnalysisReportDataExtractor-v3.zip) that was downloaded earlier
  24. Choose **Save**

## Configure network settings for the function
 25. Go to the **COnfiguration** tab and choose **VPC**
 26. Choose **Edit** and configure the following:

Action | Input 
--- | ---
VPC | Choose the option with **Cafe VPC** as the Name
Subnets | Choose the option with **Cafe Public Subnet 1** as the Name
Security Groups | Choose the option with **CafeSecurityGroup** as the Name

  27. Click **Save**

# Testing the data extractor Lambda Function
  28. Open AWS Management server on a new browser
  29. Search **Systems Manager**
  30. In the navigation pane, choose **Parameter Store**
  31. Choose the following parameter names, copy and paste the **Value** of each one

Parameter name | Value
--- | ---
/cafe/dbUrl | ec2-35-88-75-185.us-west-2.compute.amazonaws.com
/cafe/dbName | cafe_db
/cafe/dbUser | root
/cafe/dbPassword | Re:Start!9

  32. Return to the **Lambda Management Console** browser tab. On the **SalesAnalysisDataExtractor** function page, choose **Test** tab
  33. Configure the **Test event** as followed:

Action | Input
--- | ---
Test event action | Create new event
Event name | SARDETestEvent
Template | hello-world

In the **Event JSON** pane, replace the JSON object:\
```
{
  "dbUrl": "<value of /cafe/dbUrl parameter>",
  "dbName": "<value of /cafe/dbName parameter>",
  "dbUser": "<value of /cafe/dbUser parameter>",
  "dbPassword": "<value of /cafe/dbPassword parameter>"
}
```
   - Make sure to replace the values in the <> with the parameter value saved from earlier
  34. Choose **Save** and then Choose **Test**
     - It should display the message "Execution result: failed"
     - 
 ##Analyzing and correcting the Lambda function
The lambda function does not connect to the MySQL database running in a separate EC2 instance. Let's fix that!
  35. In the **Execution result** choose **Details**. Here you can analyze the error
  36. Choose the **Configuration** tab, and choose **VPC**, then click on the Security groups hyperlink
  37. In the **Security Group** page, scroll down to **Inbound Rules** and click on **Edit inbound Rules**
  38. Click on **Add rule**, and configure the following:

Action | Input
--- | ---
Type | MYSQL/ Aurora (this should automatically update Port range to `3306`
Source | Custom
On the search bar | Choose the security group with the name "CafeSecurityGroup"

  39. Click on **Save**
  40. Return to the **salesAnalysisReportDataExtractor** browser tab. Choose the **Test** tab and choose **Test** again
    - It should now run succesfully.
  41. Click on **Details** to expand it. The return should be
```
{
  "statusCode": 200,
  "body": []
}
```
