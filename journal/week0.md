# Week 0 — Billing and Architecture
**Business Scenario :**

Your company has asked to put together a technical presentation on the proposed architecture that will be implemented so it can be reviewed by the fractional CTO.
Your presentation must include a technical architectural diagram and breakdown of possible services used along with their justification.
The company also wants to generally know what spend we expect to encounter and how we will ensure we keep our spending low.

## Homework Required

### Create a new User and Generate AWS Credentials
- Go to IAM Users Console
- ``Enable console access`` for the user
- Create a new``Admin`` Group and apply ``AdministratorAccess``
- Create the user and go find and click into the user
- Click on ``Security Credentials`` and ``Create Access key``


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

### CloudShell

![CloudShell](assets/Cloudsell.PNG)

### Logical Diagram :

![Logical diagram](https://user-images.githubusercontent.com/59735117/218827213-b1082d9c-28c6-4f43-885d-8480ecdc2a44.PNG)


Here is the share link to the Logical Architectural Diagram seen in Lucid : https://lucid.app/lucidchart/14e70fc9-ab7f-47f0-956b-79569afa3ab1/edit?viewport_loc=249%2C524%2C2633%2C1155%2C0_0&invitationId=inv_3bc883e1-377c-42ce-b2a7-feb175999ccc

