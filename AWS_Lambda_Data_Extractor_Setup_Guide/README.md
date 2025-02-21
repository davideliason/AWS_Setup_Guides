# AWS Lambda Data Extractor Setup Guide

This guide walks you through creating a Lambda layer, setting up a data extractor Lambda function, configuring network settings, testing, troubleshooting, and setting up notifications using AWS SNS.

---

## Task 2: Creating a Lambda Layer and Data Extractor Lambda Function

### Task 2.1: Creating a Lambda Layer

1. **Download Required Files:**
   - [pymysql-v3.zip](#)
   - [salesAnalysisReportDataExtractor-v3.zip](#)

2. **Create Lambda Layer:**
   - Navigate to **Services > Compute > Lambda** in AWS Console.
   - Choose **Layers**.
   - Click **Create layer**.
   - Configure:
     - **Name:** `pymysqlLibrary`
     - **Description:** `PyMySQL library modules`
     - **Upload a .zip file:** Upload `pymysql-v3.zip`.
     - **Compatible runtimes:** `Python 3.9`
   - Click **Create**.

### Task 2.2: Creating a Data Extractor Lambda Function

1. In AWS Console, go to **Lambda > Functions**.
2. Click **Create function**:
   - **Author from scratch**
   - **Function name:** `salesAnalysisReportDataExtractor`
   - **Runtime:** `Python 3.9`
   - **Execution role:** Use existing role `salesAnalysisReportDERole`
3. Click **Create function**.

### Task 2.3: Adding the Lambda Layer to the Function

1. In **Function overview**, choose **Layers**.
2. Click **Add a layer**:
   - **Choose a layer:** `Custom layers`
   - **Custom layers:** `pymysqlLibrary`
   - **Version:** `1`
3. Click **Add**.

### Task 2.4: Importing the Code for the Data Extractor Lambda Function

1. Navigate to **Lambda > Functions > salesAnalysisReportDataExtractor**.
2. In **Runtime settings**, click **Edit**:
   - **Handler:** `salesAnalysisReportDataExtractor.lambda_handler`
3. Click **Save**.

4. In **Code source**:
   - Choose **Upload from > .zip file**.
   - Upload `salesAnalysisReportDataExtractor-v3.zip`.
   - Click **Save**.

### Task 2.5: Configuring Network Settings for the Function

1. Go to **Configuration > VPC**.
2. Click **Edit** and set:
   - **VPC:** `Cafe VPC`
   - **Subnets:** `Cafe Public Subnet 1`
   - **Security groups:** `CafeSecurityGroup`
3. Click **Save**.

---

## Task 3: Testing the Data Extractor Lambda Function

### Task 3.1: Launching a Test

1. Open **AWS Systems Manager > Parameter Store**.
2. Copy the following parameter values:
   - `/cafe/dbUrl`
   - `/cafe/dbName`
   - `/cafe/dbUser`
   - `/cafe/dbPassword`

3. In **Lambda Management Console**, go to the **Test** tab:
   - **Create new event**
   - **Event name:** `SARDETestEvent`
   - **Template:** `hello-world`
   - Replace JSON with:
     ```json
     {
       "dbUrl": "<value of /cafe/dbUrl>",
       "dbName": "<value of /cafe/dbName>",
       "dbUser": "<value of /cafe/dbUser>",
       "dbPassword": "<value of /cafe/dbPassword>"
     }
     ```
   - Click **Save** and then **Test**.

### Task 3.2: Troubleshooting

- If the test fails with a timeout:
  - Check **VPC > Security Groups**.
  - Ensure port **3306** is open for MySQL.
  - Add inbound rule if necessary.

### Task 3.3: Analyzing and Correcting

- After correcting the security group:
  - Retest the Lambda function.
  - A successful test returns:
    ```json
    {
      "statusCode": 200,
      "body": []
    }
    ```

### Task 3.4: Placing an Order and Testing Again

1. Access the café website:
   - Get public IP of EC2 instance.
   - Navigate to `http://publicIP/cafe`.
2. Place some orders to populate the database.
3. Retest the Lambda function.
   - The returned JSON should now include product data.

---

## Task 4: Configuring Notifications

### Task 4.1: Creating an SNS Topic

1. In AWS Console, go to **Services > Application Integration > Simple Notification Service**.
2. Choose **Topics > Create topic**.
3. Configure:
   - **Type:** `Standard`
   - **Name:** `salesAnalysisReportTopic`
   - **Display name:** `SARTopic`
4. Click **Create topic**.
5. Copy the **ARN** for later use.

### Task 4.2: Subscribing to the SNS Topic

1. Choose **Create subscription**.
2. Configure:
   - **Protocol:** `Email`
   - **Endpoint:** Enter the email address for notifications.
3. Confirm the subscription via the confirmation email.


# Sales Analysis Report Lambda Function

This guide walks you through creating, configuring, and testing the `salesAnalysisReport` AWS Lambda function, which automates the process of generating daily sales analysis reports.

## Overview

The `salesAnalysisReport` Lambda function:
- Retrieves database connection information from AWS Systems Manager Parameter Store.
- Invokes the `salesAnalysisReportDataExtractor` Lambda function to extract report data.
- Formats and publishes a message with the report data to an SNS topic.

---

## Task 5.1: Connecting to the CLI Host Instance

1. **Open** the **EC2 Management Console** and navigate to **Instances**.
2. **Select** the checkbox for the **CLI Host** instance.
3. **Click** **Connect**.
4. In the **EC2 Instance Connect** tab, click **Connect** to access the CLI Host.

---

## Task 5.2: Configuring the AWS CLI

1. In the terminal, run:
    ```bash
    aws configure
    ```

2. Provide the following details when prompted:
   - **AWS Access Key ID**: Retrieve from the **Credentials** section.
   - **AWS Secret Access Key**: Retrieve from the **Credentials** section.
   - **Default region name**: Use `us-west-2` or your appropriate region.
   - **Default output format**: Enter `json`.

---

## Task 5.3: Creating the salesAnalysisReport Lambda Function Using AWS CLI

1. Verify the presence of the Lambda code:
    ```bash
    cd activity-files
    ls
    ```

2. Retrieve the ARN of the IAM role:
   - Open the **IAM Management Console** → **Roles**.
   - Search for `salesAnalysisReportRole` and copy the **ARN**.

3. Run the following command to create the Lambda function:
    ```bash
    aws lambda create-function \
      --function-name salesAnalysisReport \
      --runtime python3.9 \
      --zip-file fileb://salesAnalysisReport-v2.zip \
      --handler salesAnalysisReport.lambda_handler \
      --region us-west-2 \
      --role <salesAnalysisReportRoleARN>
    ```

   Replace `<salesAnalysisReportRoleARN>` with your copied ARN.

---

## Task 5.4: Configuring the salesAnalysisReport Lambda Function

1. Open the **Lambda Management Console**.
2. Navigate to **Functions** → **salesAnalysisReport**.
3. Under **Configuration** → **Environment variables**:
   - Add a new environment variable:
     - **Key**: `topicARN`
     - **Value**: ARN of the SNS topic.
4. Click **Save**.

---

## Task 5.5: Testing the salesAnalysisReport Lambda Function

1. In the Lambda console, go to the **Test** tab.
2. Configure the test event:
   - **Test event action**: Create new event
   - **Event name**: `SARTestEvent`
   - **Template**: `hello-world`
3. Click **Save** and then **Test**.

### Expected Output
```json
{
  "statusCode": 200,
  "body": "\"Sale Analysis Report sent.\""
}
```

4. Check your email for a message from **AWS Notifications** titled **"Daily Sales Analysis Report"**.

---

## Task 5.6: Adding a Trigger to the Lambda Function

1. In the **Function overview**, click **Add trigger**.
2. Configure the trigger:
   - **Trigger type**: EventBridge (CloudWatch Events)
   - **Rule**: Create new rule
   - **Rule name**: `salesAnalysisReportDailyTrigger`
   - **Rule description**: `Initiates report generation on a daily basis`
   - **Rule type**: Schedule expression
   - **Schedule expression**:
     ```bash
     cron(0 20 ? * MON-SAT *)
     ```

3. Click **Add**.

---

## Conclusion

You have successfully:
- Configured IAM roles and policies.
- Created and deployed Lambda functions.
- Connected AWS CLI for Lambda management.
- Scheduled automated daily reports.
- Validated functionality with tests.

Enjoy automating your sales reports with AWS Lambda!
