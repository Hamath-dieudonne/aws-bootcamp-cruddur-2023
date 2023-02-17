# Week 0 — Billing and Architecture
**Business Scenario :**

Your company has asked to put together a technical presentation on the proposed architecture that will be implemented so it can be reviewed by the fractional CTO.
Your presentation must include a technical architectural diagram and breakdown of possible services used along with their justification.
The company also wants to generally know what spend we expect to encounter and how we will ensure we keep our spending low.

## Homework Required

## Getting the AWS CLI Working
We'll be using the AWS CLI often in this bootcamp, so we'll proceed to installing this account

### Install AWS CLI
- We are going to install When our Gitpod environment launches.
- We are going to set AWS CLI to use partial autopromt mode to make it easier to debug CLI commands.
- The bash commands we are using are the same as the AWS CLI Install Instructions

### Create a new User and Generate AWS Credentials
- Go to IAM Users Console
- ``Enable console access`` for the user
- Create a new``Admin`` Group and apply ``AdministratorAccess``
- Create the user and go find and click into the user
- Click on ``Security Credentials`` and ``Create Access key``
- Choose AWS CLI access
- Download the CSV with the credentials


![IAM User](assets/IAM_user.PNG)



### Install and verify AWS CLI
These commands allowed me to download and install AWS CLI in the gitpod environment
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```



Update my .gitpod.yml to include the following task
```yml
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial 
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

![AWS_CLI](assets/AWS_CLI.PNG)

### Set environment variables
The following examples show how you can configure environment variables for the default user.
This is not my security credentials

```
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-west-2"
```

To save our sercurity credentials in gitpod :

```
gp env AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
gp env AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
gp env AWS_DEFAULT_REGION="us-west-2"
```
### Check that the AWS CLI is working and you are the expected user
```
aws sts get-caller-identity
```
The reason I did this was to make sure I am who I say I am, because when you start having multiple logins and multiple profiles, you can run things in the wrong place without realizing it.

You should see something like this :

![aws sts get-caller-identity](assets/get.PNG)

You can also do that in CloudSell

![CloudShell](assets/Cloudsell.PNG)

### Logical Diagram :

![Logical diagram](https://user-images.githubusercontent.com/59735117/218827213-b1082d9c-28c6-4f43-885d-8480ecdc2a44.PNG)


[Shared Lucid Diagramm link](https://lucid.app/lucidchart/14e70fc9-ab7f-47f0-956b-79569afa3ab1/edit?viewport_loc=249%2C524%2C2633%2C1155%2C0_0&invitationId=inv_3bc883e1-377c-42ce-b2a7-feb175999ccc)

### Enable Billing
We need to turn on Billing Alerts to receive alerts

- In your Root Account go to the Billing Page
- Under ``Billing Preferences`` Choose ``Receive Billing Alerts``
- Save Preferences

![free tier alert](assets/free tier alert.PNG)

## Creating a Billing Alarm
### Create SNS Topic
- We need an SNS Topic before we create an alarm
- The SNS topic is what will delivery us an alert when we get overdilled

We'll create a SNS Topic

```
aws sns create-topic --name billing-alarm
```
which will return a TopicARN
We'll create a subscription supply the TopicARN and our Email

```
aws sns subscribe \
    --topic-arn TopicARN
    --protocol email\
    --notification-endpoint your@email.com
```
Check your email and confirm the subscription

### Create Alarm
- aws cloudwatch put-metric-alarm
- Create an Alarm via AWS CLI
- We need to update the configuration json script with the TopicARN we generated earlier
- We are just a json file beacause --metrics is required for expressions and so it's easier to us a JSON file


## Create an AWS Budget
I foolowed instructions on AWS [CLI Documentation page](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html#examples) to create my budget

```json
aws budgets create-budget \
    --account-id 111122223333 \
    --budget file://budget.json \
    --notifications-with-subscribers file://notifications-with-subscribers.json
```

Contents of ``budget.json``:

```json
{
    "BudgetLimit": {
        "Amount": "10",
        "Unit": "USD"
    },
    "BudgetName": "Example Tag Budget",
    "BudgetType": "COST",
    "CostFilters": {
        "TagKeyValue": [
            "user:Key$value1",
            "user:Key$value2"
        ]
    },
    "CostTypes": {
        "IncludeCredit": true,
        "IncludeDiscount": true,
        "IncludeOtherSubscription": true,
        "IncludeRecurring": true,
        "IncludeRefund": true,
        "IncludeSubscription": true,
        "IncludeSupport": true,
        "IncludeTax": true,
        "IncludeUpfront": true,
        "UseBlended": false
    },
    "TimePeriod": {
        "Start": 1477958399,
        "End": 3706473600
    },
    "TimeUnit": "MONTHLY"
}
```

Contents of ``notifications-with-subscribers.json``:

```json
[
    {
        "Notification": {
            "ComparisonOperator": "GREATER_THAN",
            "NotificationType": "ACTUAL",
            "Threshold": 80,
            "ThresholdType": "PERCENTAGE"
        },
        "Subscribers": [
            {
                "Address": "example@example.com",
                "SubscriptionType": "EMAIL"
            }
        ]
    }
]
```
Back to the budgets.
Here is our Example Tag Budget that has been set at $10.

![Budget](assets/Budgets.PNG)

Billing Alarm
So when you create an SNS topic and add a notification endpoint like email, you have to confirm it, otherwise it won't work.
![Billing Alarm](assets/billing alarm.PNG)

- DailyEstimateCharge Alarm Code
![DailyEstimateCharge](assets/alarmconfig.PNG)

- DailyEstimateCharge Alarm proof
![cloudwatch alarm](assets/cloudwatch alarm.PNG)










