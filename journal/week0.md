# Week 0 — Billing and Architecture
## Creating IAM user and getting the AWS CLI Working

We'll be using the AWS CLI often in this bootcamp,
so we'll proceed to installing this account.


### Create a new User and Generate AWS Credentials

- Go to (IAM Users Console] and create a new user
- `Enable console access` for the user
- `Enable Users must create a new password at next sign-in` for new user
- Create a new `Admin` Group and apply `AdministratorAccess`
- Enable MFA for the root and IAM user
- Click on `Security Credentials` and `Create Access Key`
- Choose AWS CLI Access
- Download the CSV with the credentials for future use

### Install AWS CLI

- Install the AWS CLI when our Gitpod enviroment lanuches for easy access to aws console
- Set AWS CLI to use partial autoprompt mode by using cmd: --cli-auto-prompt
- The bash commands we are using are the same as the [AWS CLI Install Instructions]https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html


Update the `.gitpod.yml` to include the following task. We will add 3 commands to install AWS module in our gitpod env
- `curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"` fetch the zip file for aws installation from aws website
- `unzip awscliv2.zip` unzip the zip file for aws mod
- `sudo ./aws/install`  install aws using sudo permissions

```sh
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

We'll also run these commands indivually to perform the install manually


### Set Env Vars

We will set these credentials for the current bash terminal
```
export AWS_ACCESS_KEY_ID=""
export AWS_SECRET_ACCESS_KEY=""
export AWS_DEFAULT_REGION=ca-central-1
```

`gp env` is used to make all settings reboot persistent, it means if we reboot the gitpod it will recall user's access keys to easy terminal access.
```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=ca-central-1
```

### Check that the AWS CLI is working and you are the expected user

```sh
aws sts get-caller-identity
```

You should see something like this:
```json
{
    "UserId": "AIFBZRJIQN2ONP4ET4EK4",
    "Account": "655602346534",
    "Arn": "arn:aws:iam::655602346534:user/G31"
}
```

## Enable Billing 

Steps to turn on Billing Alerts to recieve alerts...


- In your Root Account go to the [Billing Page](https://console.aws.amazon.com/billing/)
- Under `Billing Preferences` Choose `Receive Billing Alerts`
- Save Preferences


## Creating a Billing Alarm

### Create SNS Topic

- Create SNS topic before we create an alarm.
- The SNS topic is what will delivery us an alert when we get overbilled
- [aws sns create-topic](https://docs.aws.amazon.com/cli/latest/reference/sns/create-topic.html)

Steps to create a SNS Topic
```sh
aws sns create-topic --name billing-alarm
```
which will return a TopicARN

Steps to create a subscription supply the TopicARN and our Email
We will use `arn value`arn:aws:iam::655602346534:user/G31 and our email for notification endpoint
```sh
aws sns subscribe \
    --topic-arn TopicARN \
    --protocol email \
    --notification-endpoint your@email.com
```

Check your email and confirm the subscription

#### Create Alarm

- [aws cloudwatch put-metric-alarm](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)
- [Create an Alarm via AWS CLI](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/)
- We will update the configuration json script with the TopicARN we generated earlier
- We are just a json file because --metrics is is required for expressions and so its easier to us a JSON file.

```sh
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```

## Create an AWS Budget

[aws budgets create-budget](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html)

Get your AWS Account ID
```sh
aws sts get-caller-identity --query Account --output text
```

- Supply your AWS Account ID
- Update the json files
- This is another case with AWS CLI its just much easier to json files due to lots of nested json

```sh
aws budgets create-budget \
    --account-id AccountID \
    --budget file://aws/json/budget.json \
    --notifications-with-subscribers file://aws/json/budget-notifications-with-subscribers.json
```

## Logical architecture 

I prepared a logical diagram of the application architecture. The diagram contains major AWS services required for the fucntioning of the app.
I used arrows to depict data flow across several AWS services. The diagram is as follows:
![alt text](https://github.com/Gul31/aws-bootcamp-cruddur-2023/blob/main/_docs/assets/Cruddur-Logical%20Diagram.png)

The diagram has the following services:
- **Client:** any user can access the application over internet
- **Authentication:** Authentication of the user will be done prior accessing the app.
- **Route 53:** DNS service from AWS. 
- **Application Load Balancer:** This load balancer is used to maintain traffic flow to avoid congestion or crashing.
- **Virtual Private Network:** VPC provides private network capabilities to the cloud env.
- **ECS EC2 Container:** it provides containerization env for our application
- **Frontend:** it depicts the frontend portion of our app.
- **Backend:** it depicts the backend portion of our app.
- **REST API:** it is used for API calls for communication between Frontend and Backend.
- **AWS DynamoDB:** it is non-relational database to store app data.
- **AWS RDS:** it is relational database to store app data.
- **AppSync:** it allows your applications to access exactly the data they need via API. it is used to create and manage APIs.
- **Momento:** it is used to serverless caching service.


## Security Considerations

Here are some security considerations for the week0:
- `Enabled MFA` for IAM user and root account.
- Stopped using root account for regular activtites.
- Created an `Organizational Unit` for easier management of multiple AWS accounts.
- Following `Least priviledge principles`.
- Enabling `CloudTrail` to log management events for AWS account. We can also use it at OU level to log multiple AWS accounts.
