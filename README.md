**ATTENTION:** This is currently a work in progress. Please check back when the course is complete

![](./images/partner_program.png)

# Overview
Hello and welcome to this Dynatrace Partner TechLab. We are launching these as self-paced training for all our partners. During this session we will focus on automation and complete hands on exercises on how to automate the configuration of a monitoring environment.

The goals of this tutorial are;
1. **Tagging** - Create tagging rules via the API. Tags are useful in finding/identifying componets, management zone configuration, alerting profiles and differentiating between environments.
2. **Management Zones** - Automatically create management zones in your environment via the API. Useful for splitting environments.
3. **Web Applications** - Automatically create web applications in your enviroment via the API. Useful for onboarding new RUM applications.
4. **Application Detection Rules** - Automatically create application detection rules via the API. Useful for to ensure your RUM traffic is assigned to the correct web application.
5. **Request Attributes** - Automatically create request attributes rules via the API. Useful to extract additional information that may be of use in you system such as loyalty status or revenue.
6. **Load Test Integration** - Automatically create request attributes rules that would enable integration with Load Testing tools via the API. Useful to integrate with your load testing tools.
7. **Calculated Service Metrics** - Automatically create calculated service metric rules via the API. Useful to configre additional metrics to be calculated, For example, a ride-hailing company might want to keep track of the number of active drivers per region, or the average ride distance, and be alerted on significant anomalies.
8 **Modify Exiting Web Application** - Automatically modify web application settings via the API. Useful to set custom Appdex or enable additional features such as javaScript Framework Support or conversion goals.
9. **Alerting Profiles** - Automatically create alerting profiles via the API. Useful to filter alerts such as for different environemnts like Production and Test.
10. **Notifications** - Automatically create alerting profiles via the API. Useful to configure integrations with incident management platforms.

We will run these steps individually so you get a feel for how they operate. In a real life scenario you could run them all one after the other.

## Pre-Requisites
1. You need an AWS account. If you don't have one [get one here](https://aws.amazon.com/)
2. AWS Access key and secret access key for an IAM user with rights ec2:RunInstances,  ec2:DescribeImages & ec2:CreateTags. If you don't know how to do this don't worry it is explained in the next section.
3. You need a Dynatrace instance. The instance can be SaaS or Managed.
4. An API token with the following roles - 
![](./images/api.png)
5. Optional: ActiveGates - An ActiveGate is not required to complete these exercises, but depending on your install and security requirements you may also require a Cluster and/or Environment ActiveGate. For more info see [When do I need to install an ActiveGate?](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-activegate/basic-concepts/when-do-i-need-to-install-an-activegate/)
6. [Postman](https://www.postman.com/downloads/) installed
7. You need to clone or copy the content of this GitHub repo to your local disk!

## Preparation

**Amazon Web Services (AWS)**

**ATTENTION:** This is the same as the tokens created in TechLab-Environment-Automation so if you are following on from that, you still have the credentials and they are still active in EC2 you can use thoses and do not need to complete this.

We are going to use AWS EC2. We have tested this tutorial on eu-west-1 (Ireland), Paris (eu-west-3) & ..... . To be on the safe side we suggest you pick one of these regions!

Ensure you have created an IAM user  with rights ec2:RunInstances, ec2:DescribeImages & ec2:CreateTags. We recommend you create a user for this with only these rights that can be removed in the future.

Login to your AWS account and navigate to IAM > Users > Add user and complete the following
1. Give your user a name and select Programmatic access

![](./images/preparation/awsconfigure.png)

2. Select Create Group

![](./images/preparation/awsCreateGroup.png)

3. Name your group and select Create policy. The Policy creater should open in a new tab.

![](./images/preparation/awsGroupName.png)

4. Choose the service ec2 and under actions search and add RunInstances, DescribeImages & CreateTags. Then click Review policy

![](./images/preparation/awsCreatePolicy.png)

5. It should look like the following. Select Create Policy

![](./images/preparation/awsPolicy.png)

6. Return to your previous tab, click Refresh and search for you new policy. Tick the policy and select Create Group

![](./images/preparation/awsPolicyGroup.png)

7. Ensure your group has been added for your user and the policy is attached. Click Next:tags. Tags are optional so add one if you wish and click Next:Review

![](./images/preparation/awsSetPer.png)

8. Your new user should look like the following. Click Create User. Store your credentials, you can download csv to save them in a file. 

![](./images/preparation/awsUserSum.png)

Once you have your IAM user and role we need a couple of things
1. Your AWS Access Key ID. It should look something like `AKIAIOSFODNN7EXAMPLE`
2. Your AWS Secret key. It should look something like `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
3. Your AWS region. It should look something like `eu-west-1`

**Dynatrace**

You will need a SaaS or Managed Environment for this course. If you have previously completed TechLab-Environment-Automation you can use this.

Once you have your environment we need a couple of things
1. Your dynatrace URL. It should look something like for SaaS - `{your-environment-id}.live.dynatrace.com` or for Managed - `{your-domain}/e/{your-environment-id}`
2. An API token. Inside your environment go to Settings -> Integration -> Dynatrace API and select Generate token. Grant it the scopes of Access problems and event feed, metrics, and topology; Read configuration; Write configuration; Capture request data; 

![](./images/preparation/clusterapi.png)

Expand the token and copy the API key

![](./images/preparation/clusterapitoken.png)

**Postman**

For this tutorial we will leverage postman to make our API calls. If you would like to take these concepts and incorporate them into your dynatrace provisioning process you can choose any tool you like providing it can make API calls. 

Once you have installed Postman we need a couple of things
1. Upload the <a id="raw-url" href="https://raw.githubusercontent.com/dynatrace-partners/TechLab-Environment-Automation/master/postman/TechLab-Environment-Automation.postman_environment.json">postman environment</a> we have provided.

    Inside Postman click on import, select file and upload TechLab-Environment-Automation.postman_environment.json

    ![](./images/preparation/postmanEnv.png)

2. Upload the [<a id="raw-url" href="https://raw.githubusercontent.com/dynatrace-partners/TechLab-Environment-Automation/master/postman/TechLab-Environment-Automation.postman_collection.json">postman collection</a> we have provided.

    Repeat the same process to import TechLab-Environment-Automation.postman_collection.json

    ![](./images/preparation/postmanCollection.png)

3. Configure your postman environment variables

    * Click on the ![](./images/preparation/manageEnvironments.png) to manage your environments

    * Click on the environment name TechLab-Environment-Automation

    * Set the first 5 environment variables to your values. Ensure to set both the initial and current values. For dtManaged it should look like 'xxxxxx.dynatrace-managed.com' do not include the 'https://'. Do not tough the 'installET' variable.

    ![](./images/preparation/environmentVars.png)

    * Click on update to save your changes

4. Select your postman environment in the upper right hand corner dropdown

    ![](./images/preparation/postmanSelEnv.png)

**ATTENTION:** The first 3 requests create the same instances as in TechLab-Environment-Automation so if you are following on from that, your ec2 instances are still running and you plan to use those, then you can skip the first 3 steps.

# 1. Creating a dynatrace environment API token (Optional)

To create a dynatrace environment API token ensure you have completed the creating a new monitoring environment exercise.

## Creating a dynatrace environment API token background 

**What is a dynatrace environment API token?**

To get authenticated to use the Dynatrace API, you need a valid API token. Access to the API is fine-grained, meaning that you also need the proper permissions assigned to the token. In our example we want to automatically install the OneAgent on an ec2 host, so we need to create a token with the InstallerDownload permission.

**When would you create a dynatrace environment API token?**

An environment API token is required when you wish to access the dynatace APIs for a given environment. You can assign multiple permissions to a single token, or you can generate several tokens, each with different access levels and use them accordingly—check your organization's security policies for the best practice.

**How do you create a dynatrace environment API token?**

API tokens can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create a new API token with InstallerDownload permissions so we can automate the installation of the OneAgent when we start our new EC2 instance.

## Creating a dynatrace environment API token configuration

**Request configuration**

**ATTENTION:** There is currently no tokens V2 environment API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the environment v1 API endpoint tokens. This is different than the previous endpoints as this is now and environment endpoint where we have previously used cluster endpoints. Environment endpoints apply specifically to the specified environment. You will notice in the URL we are now specifying an environment in the parameter \{\{dtURL\}\}, that you should have set during the preparation. 

By making a request to this API endpoint we will create a new API token on our new environment with the rights to download the OneAgent Installer. For security we will set the token to expire in 4 hours. This means you need to create your ec2 instance within this time.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{envTokenManagementToken\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{envTokenManagementToken\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server.
Key | Value | Description
------------ | ------------- | -------------
name | TechLab-Managed-Cluster-Automation | This will be the name of our token. Dynatrace doesn't enforce unique token names. You can create multiple tokens with the same name. Be sure to provide a meaningful name for each token you generate. Proper token naming helps you to efficiently manage your tokens and perhaps delete them when they're no longer needed.
scopes | InstallerDownload | This is the permissions the token will hold. In our case we only require the installer download. For a full list of permissions see [token permission](https://www.dynatrace.com/support/help/dynatrace-api/basics/dynatrace-api-authentication#token-permissions)
expiresIn | 4 HOURS | This is an optional parameter that will cause the token to expire after a given time. It is strongly recommended to rotate your tokens frequently and this parameter can help you with that, but be careful when using integrations. If not set then the token never expires.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the new API token and sets it as environment variable paasToken so we can use it in the Launch AWS easyTravel Instances request to auto deploy the OneAgent.

**Executing the request**
1. Open the Create Installer Token request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createInstTokResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check that the paasToken environment variable has been set in postman

![](./images/envtoken/createInstTokVars.png)

Congratulations you have just created a new API token with installer download permission via an API call. Now Let's launch some EC2 instances and automatically deploy the OneAgent.

# 2. Get AWS AMI ID (Optional)

As this is not a dynatrace API call and is purely to get an AMI ID we will not cover this in as much detail. For more information on it please see the [aws documentation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html). This request polls the AWS API to get the latest Ubuntu AMI in your region, it is filtered to the Ubuntu Cloud Account. This way when you launch an ec2 instance in the next step it will use this AMI to ensure you are on an up-to-date version. When you send the request the AMI ID will be stored as environment variable ImageId.

**Executing the request**
1. Open the Get AWS AMI ID request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/ami/getAMIResp.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

5. Check that the ImageId environment variable has been set in postman

![](./images/ami/getAMIVars.png)

Now that we have the latest Ubunntu image AMI we can launch our ec2 instances.

# 3. Launching easyTravel ec2 Instances and auto deploying the OneAgent (Optional)

To launch the easytravel ec2 instances ensure you have created your environment, installer token and got the AWS AMI for the latest Ubuntu release. 
As this is not a dynatrace API call and is used to start some ec2 we will not cover this in as much detail, but we will cove how it auto deploys the OneAgent. For more information on it please see the [aws documentation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RunInstances.html)

## Launching easyTravel ec2 Instances and auto deploying the OneAgent 

**How do you launch an easyTravel ec2 Instances and auto deploying the OneAgent?**

We will leverage the aws RunInstances API call to start 2 instances of the latest Ubuntu AMI that we gathered in the previous call. This will be a blank Ubuntu image so we will leverage [AWS UserData](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to install and configure both easyTravel and the OneAgent. The UserData is sent as base64-encoded text, but if you want to see the commands that are executed I have provided a sample script [here](./aws/userData.txt). This sample is not exactly the same as the Postman execution as we dynamically set some values in the postman execution we will pass the correct dynatrace environment and API token details and we will set a host group. We will execute this request twice, the first will create an instance with the host group production and the second will create another instance with the host group test.

## Launching easyTravel ec2 Instances and auto deploying the OneAgent configuration

**Request configuration**

**ATTENTION:** There are some values that are unique to your ec2 account that you will have to set for this request to work.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the aws API endpoint to execute the action runInstances. This API call will start an instance of the latest Ubuntu image from the AMI that was gathered in the previous step.

**Parameters**

There are a lot of Parameters in this request, but I will only cover the ones relevant to dynatrace or that you may need to change for your environment.

You may need to set the following Parameters. Determine if you need to set them by reading the table below. If you do need to set them then add the value in postman and ensure you tick the box to the left of the key name to enable it. 
Key | Value | Requirement | Description
------------ | ------------- | ------------- | -------------
KeyName | The name of your key pair | Optional | The name of the key pair used to connect to your instance. You can create a key pair using [CreateKeyPair](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateKeyPair.html). If you do not specify a key pair, you can't connect to the instance. In our use case there is no direct requirement to access the environment, but you may find it useful if you want to perform troubleshooting or use the instance in the future for other things.
SecurityGroupId | The ID of your security group | Optional | If you don't specify a security group ID, aws use the default security group. For more information, see [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html). Depending on your setup this may be required. A security group acts as a virtual firewall for your instance to control incoming and outgoing traffic so you may require a specific one to allow the correct access to and from your instance. At a minimum you will need inbound on 9080 & 8079 to access the easyTravel & easyTravel Angular front ends and outboun either 443 for direct OneAgent communication or 9999 via an ActiveGate. 

The secret sauce here that you should be aware of is [AWS UserData](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html). When we launch this request we supply a set of commands to be executed on the ec2 instance at start up. The userdata is preconfigured, do not make changes or it may no longer work. In our case these commands will do the following;
1. Updates the package lists by running apt-get update
2. Installs the required packages
3. Downloads and installs the latest version of easyTravel
4. Makes required config changes to easyTravel including API call to set correct public DNS of instance for the source of the traffic that is generated.
5. Installs the OneAgent for your newly created environment leveraging the API token you created in exercise 5
6. Starts the easyTravel angular app. This means you don't even need to access the environment.

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct host group when requests are executed and updates the userdata to be sent to aws.

**ATTENTION:** When we create these hosts they are set to auto terminate after 8 hours. If you do not want them to auto terminate then please comment out lines 34 & 35. The host is also set to auto terminate on shutdown so keep in mind even if you comment this line out the host would still terminate if you shut it down.

To change the shutdown behaviour of an instance using the console (only after you have started your instance)

1. Open the Amazon EC2 console at `https://console.aws.amazon.com/ec2/`.
2. In the navigation pane, choose Instances.
3. Select the instance, and choose Actions, Instance Settings, Change Shutdown Behaviour. The current behaviour is already selected.
4. To change the behaviour, select an option from the Shutdown behaviour list, and then choose Apply.
![](./images/runinstances/shutdown_behavior_dialog.png)


**Executing the request**
This request is designed to start 2 instance of easyTravel, one with the host group production and the second with the host group test. We will execute the request twice to achieve this. No changes are required as they are done automatically
1. Open the Launch AWS easyTravel Instances request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response and the Production instance name tag.

![](./images/runinstances/runInstancesRespProd.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

5. Execute the request again to start the second instance.
6. Check that the request received a `200 OK` response and the Test instance name tag.

![](./images/runinstances/runInstancesRespTest.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

7. Inside your AWS console check the instances have been created

![](./images/runinstances/runInstancesEC2.png)

8. Check the deployment status in dynatrace to see if your hosts are now monitored. Please note this can take 5-10 mins. 

![](./images/runinstances/runInstancesDT.png)

Congratulations you have just created 2 new ec2 instances and auto deployed the OneAgent.

**Troubleshooting**
If after 10 mins your hosts have not appeared in dynatrace check the following.
1. In the AWS EC2 console select one of your instances and click on Actions > Instance settings > View/Change User Data.
    * Verify the userdata has not been corrupted
    * Check the values for your managed environment URL add API token in the wget command
2. Log on to your instance and view log /var/log/cloud-init-output.log to check for errors in downloading, installing or connection of your OneAgent. You can do this by executing `Tail -f /var/log/cloud-init-output.log`. Please note to access the instance you will need to have specified KeyName when you created the instance. 

