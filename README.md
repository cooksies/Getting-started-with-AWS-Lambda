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
  10. Download these lab files to your local machines.
      [pymysql-v3.zip](https://github.com/cooksies/Getting-started-with-AWS-Lambda/blob/main/pymysql-v3.zip)
      [salesAnalysisReportDataExtractor-v3.zip](
