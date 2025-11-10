# AWS-Weekly-Cost-Report-Automation
<h2>High-Level Overview:</h2>

<h4>This project showcases an automated AWS cost monitoring system that tracks spending, sends threshold alerts, and generates weekly spend reports using Lambda, EventBridge, Cost Explorer, and SNS. Also, storing reports within an S3 Bucket for later retrieval.</h4>

1) CloudWatch Billing Alarms track AWS spending automatically.

2) When costs exceed set thresholds ($50, $100, $150, $200), EventBridge triggers an alert.

3) SNS (Simple Notification Service) sends email or SMS notifications to subscribers.

4) Every 7 days, EventBridge triggers a Lambda function.

5) Lambda uses the Cost Explorer API to generate a weekly cost report. The report is:

- Emailed to subscribers via SNS, and

- Stored in Amazon S3 for recordkeeping.


<h2>Impact & Relevance</h2>

- Shows cloud automation skills

- Demonstrates cost visibility & compliance (huge in DoD/federal contexts)

- Uses serverless + event-driven design

<h2>Architecture Diagram</h2>

<img width="600" height="936" alt="image" src="https://github.com/user-attachments/assets/a6229518-779e-4415-b617-3f11b22f08f3" />

<h2>Sample Output</h2>

<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/ae16bc17-c765-42f8-afb6-5ea9bab98298" />

<h4>AWS Weekly Spend Report (2025-09-30 to 2025-10-07):</h4>

- 2025-09-30: $12.34

- 2025-10-01: $15.67

- Total: $98.45

<h2>Skills Demonstrated</h2>

- Cloud cost tracking and budgeting automation.

- Event-driven architecture using AWS serverless services.

- Real-time alerting and scheduled reporting with CloudWatch, EventBridge, and Lambda.

- Integration of multiple AWS services with IAM security best practices.

- Efficient cost monitoring under the AWS Free Tier for hands-on learning and portfolio demonstration.

<h2>CloudShell Code Used</h2>

import boto3

import datetime

import os

def lambda_handler(event, context):
    # Initialize Cost Explorer and SNS clients
    ce = boto3.client('ce')
    sns = boto3.client('sns')
 
    # Define the time range: last full week (Monday–Sunday)
    end = datetime.date.today()
    start = end - datetime.timedelta(days=7)
 
    # Query Cost Explorer for total unblended cost
    response = ce.get_cost_and_usage(
        TimePeriod={
            'Start': start.strftime('%Y-%m-%d'),
            'End': end.strftime('%Y-%m-%d')
        },
        Granularity='DAILY',
        Metrics=['UnblendedCost']
    )
 
    # Build a readable report
    report_lines = []
    for result in response['ResultsByTime']:
        amount = result['Total']['UnblendedCost']['Amount']
        date = result['TimePeriod']['Start']
        report_lines.append(f"{date}: ${float(amount):.2f}")
 
    report = "AWS Weekly Spend Report\n\n" + "\n".join(report_lines)
 
    # Publish to SNS
    sns.publish(
        TopicArn=os.environ['SNS_TOPIC_ARN'],
        Subject="Weekly AWS Spend Report",
        Message=report
    )
 
    return {
        'statusCode': 200,
        'body': f"Report sent to SNS topic {os.environ['SNS_TOPIC_ARN']}"
    }
 
