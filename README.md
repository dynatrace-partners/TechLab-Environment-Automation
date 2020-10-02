**ATTENTION:** This is currently a work in progress. Please check back when the course is complete

![](./images/partner_program.png)

# Overview
Hello and welcome to this Dynatrace Partner TechLab. We are launching these as self-paced training for all our partners. During this session we will focus on automation and complete hands on exercises on how to automate the configuration of a monitoring environment.

The goals of this tutorial are;
1. **Tagging** - Create tagging rules via the API. Tags are useful in finding/identifying components, management zone configuration, alerting profiles and differentiating between environments.
2. **Management Zones** - Automatically create management zones in your environment via the API. Useful for splitting environments.
3. **Web Applications** - Automatically create web applications in your envirnoment via the API. Useful for onboarding new RUM applications.
4. **Application Detection Rules** - Automatically create application detection rules via the API. Useful for to ensure your RUM traffic is assigned to the correct web application.
5. **Request Attributes** - Automatically create request attributes rules via the API. Useful to extract additional information that may be of use in your system such as loyalty status or revenue.
6. **Load Test Integration** - Automatically create request attributes rules that would enable integration with Load Testing tools via the API. Useful to integrate with your load testing tools.
7. **Calculated Service Metrics** - Automatically create calculated service metric rules via the API. Useful to configure additional metrics to be calculated, for example, a ride-hailing company might want to keep track of the number of active drivers per region, or the average ride distance, and be alerted on significant anomalies.
8 **Modify Exiting Web Application** - Automatically modify web application settings via the API. Useful to set custom Appdex or enable additional features such as javaScript Framework Support or conversion goals.
9. **Alerting Profiles** - Automatically create alerting profiles via the API. Useful to filter alerts such as for different environments like Production and Test.
10. **Notifications** - Automatically create alerting profiles via the API. Useful to configure integrations with incident management platforms.

We will run these steps individually, so you get a feel for how they operate. In a real-life scenario you could run them all one after the other.

## Pre-Requisites
1. You need an AWS account. If you don't have one [get one here](https://aws.amazon.com/)
2. AWS Access key and secret access key for an IAM user with rights ec2:RunInstances,  ec2:DescribeImages & ec2:CreateTags. If you don't know how to do this don't worry it is explained in the next section.
3. You need a Dynatrace instance. The instance can be SaaS or Managed.
4. An API token with the following roles - 
![](./images/api.png)
5. Optional: ActiveGates - An ActiveGate is not required to complete these exercises but depending on your install and security requirements you may also require a Cluster and/or Environment ActiveGate. For more info see [When do I need to install an ActiveGate?](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-activegate/basic-concepts/when-do-i-need-to-install-an-activegate/)
6. [Postman](https://www.postman.com/downloads/) installed
7. You need to clone or copy the content of this GitHub repo to your local disk!

## Preparation

**Amazon Web Services (AWS)**

**ATTENTION:** These are the same as the tokens created in TechLab-Managed-Cluster-Automation so if you are following on from that, you still have the credentials and they are still active in EC2 you can use those and do not need to complete this.

We are going to use AWS EC2. We have tested this tutorial on eu-west-1 (Ireland), Paris (eu-west-3) & ..... . To be on the safe side we suggest you pick one of these regions!

Ensure you have created an IAM user  with rights ec2:RunInstances, ec2:DescribeImages & ec2:CreateTags. We recommend you create a user for this with only these rights that can be removed in the future.

Login to your AWS account and navigate to IAM > Users > Add user and complete the following
1. Give your user a name and select Programmatic access

![](./images/preparation/awsconfigure.png)

2. Select Create Group

![](./images/preparation/awsCreateGroup.png)

3. Name your group and select Create policy. The Policy creator should open in a new tab.

![](./images/preparation/awsGroupName.png)

4. Choose the service ec2 and under actions search and add RunInstances, DescribeImages & CreateTags. Then click Review policy

![](./images/preparation/awsCreatePolicy.png)

5. It should look like the following. Select Create Policy

![](./images/preparation/awsPolicy.png)

6. Return to your previous tab, click Refresh and search for your new policy. Tick the policy and select Create Group

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
2. An API token. Inside your environment go to Settings -> Integration -> Dynatrace API and select Generate token. Grant it the scopes of Access problems and event feed, metrics, and topology; Read configuration; Write configuration; Capture request data;  Token management and Read entities using API V2.

![](./images/preparation/dtapi.png)

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

    * Set the first 5 environment variables to your values. Ensure to set both the initial and current values. For dtManaged it should look like 'xxxxxx.dynatrace-managed.com' do not include the 'https://'. Do not touch the 'installET' variable.

    ![](./images/preparation/environmentVars.png)

    * Click on update to save your changes

4. Select your postman environment in the upper right-hand corner dropdown

    ![](./images/preparation/postmanSelEnv.png)

**ATTENTION:** The first 3 requests create the same instances as in TechLab-Managed-Cluster-Automation so if you are following on from that, your ec2 instances are still running and you plan to use those, then you can skip the first 3 steps.

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
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server.
Key | Value | Description
------------ | ------------- | -------------
name | TechLab-Environment-Automation | This will be the name of our token. Dynatrace doesn't enforce unique token names. You can create multiple tokens with the same name. Be sure to provide a meaningful name for each token you generate. Proper token naming helps you to efficiently manage your tokens and perhaps delete them when they're no longer needed.
scopes | InstallerDownload | This is the permissions the token will hold. In our case we only require the installer download. For a full list of permissions see [token permission](https://www.dynatrace.com/support/help/dynatrace-api/basics/dynatrace-api-authentication#token-permissions)
expiresIn | 4 HOURS | This is an optional parameter that will cause the token to expire after a given time. It is strongly recommended to rotate your tokens frequently and this parameter can help you with that but be careful when using integrations. If not set then the token never expires.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the new API token and sets it as environment variable `paasToken` so we can use it in the Launch AWS easyTravel Instances request to auto deploy the OneAgent.

**Executing the request**
1. Open the `Create Installer Token \(Optional\)` request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createInstTokResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check that the paasToken environment variable has been set in postman

![](./images/envtoken/createInstTokVars.png)

If you wish to see the token in your dynatrace environment you can see it under Deploy Dynatrace > Set up PaaS integration

![](./images/envtoken/createInstTokDT.png)

Congratulations you have just created a new API token with installer download permission via an API call. Now Let's launch some EC2 instances and automatically deploy the OneAgent.

# 2. Get AWS AMI ID (Optional)

As this is not a dynatrace API call and is purely to get an AMI ID we will not cover this in as much detail. For more information on it please see the [aws documentation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html). 

This request polls the AWS API to get the latest Ubuntu AMI in your region, it is filtered to the Ubuntu Cloud Account. This way when you launch an ec2 instance in the next step it will use this AMI to ensure you are on an up-to-date version. When you send the request the AMI ID will be stored as environment variable ImageId.

**Executing the request**
1. Open the `Get AWS AMI ID \(Optional\)`request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/ami/getAMIResp.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

5. Check that the ImageId environment variable has been set in postman. Please note it may be different than my screen shot as this will change every time a new image is realeased.

![](./images/ami/getAMIVars.png)

Now that we have the latest Ubuntu image AMI we can launch our ec2 instances.

# 3. Launching easyTravel ec2 Instances and auto deploying the OneAgent (Optional)

To launch the easyTravel ec2 instances ensure you have created your environment, installer token and got the AWS AMI for the latest Ubuntu release. 
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
SecurityGroupId | The ID of your security group | Optional | If you don't specify a security group ID, aws use the default security group. For more information, see [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html). Depending on your setup this may be required. A security group acts as a virtual firewall for your instance to control incoming and outgoing traffic so you may require a specific one to allow the correct access to and from your instance. At a minimum you will need inbound on 9080 & 8079 to access the easyTravel & easyTravel Angular front ends, inbound 8091 to access the easyTravel REST API and outbound either 443 for direct OneAgent communication or 9999 via an ActiveGate. 

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
1. Open the `Launch AWS easyTravel Instances \(Optional\)` request.
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

# 4. Get host information

Now that you have deployed your two hosts, log into dyantrace and check out the monitoring. Is it easy to tell which host is test and which is production? 

![](./images/runinstances/hostsDT.png)

Te only real way to currently tell which is which is to look at the host information where we can see our host group. This might not be that much of an issue in our environment as we only have 2 hosts but in a real customer environment this would be challenging. You will also see here the additional meta data we supplied when we created the hosts which we will use later.

![](./images/runinstances/hostPropsDT.png)

How would you filter to only see the data for one environment? Have a look at the Transactions and services tab. You should see that each service exists twice. This is because the host group automatically separates the process groups and services. 

![](./images/runinstances/servicesDT.png)

In these next requests we want to configure our environment so the users can get better value from it. In order to do this we need some information on our hosts so rather than gather that manually we will use the API to get the info we need and then use it to configure the environment using API calls.

## Get host information background 

**What is the hosts API?**

The hosts API gets the list of all hosts in your Dynatrace environment, along with their parameters.

**When would you call the hosts API?**

You would call this API if you wanted info on your hosts. This could be to audit what is running, use the info to update a CMDB or in our case get info on the hosts to use in the Dynatrace configuration, specifically the public DNS names of our hosts which we will use in our application detection rules.

**How do you call the hosts API?**

The hosts API can be called by anything that can make an API request like postman, curl or indeed the API explorer built into dynatrace.

## Get host information configuration

**Request configuration**

**ATTENTION:** There is currently is a v2 version of this API \(although not identical\) but currently it cannot access AWS properties of hosts, once this is added we will migrate it.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the entities v1 API endpoint hosts. You will notice in the URL we are now specifying an environment in the parameter \{\{dtURL\}\}, that you should have set during the preparation. We are also specifying  the url parameter  relativeTime=min, this will poll the API for all the hosts in the last 1 minute.

By making a request to this API endpoint will get a list of all hosts in the last one minute and their properties. Please note the request finds the hosts based on the hostgroups Production and Test, if you have other hosts in the groups it may not get the details of the correct hosts.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the public host names of both hosts and sets them as environment variables prodPublicHostName & testPublicHostName so we can use it in the Create App Detection rules request to separate our production and test RUM traffic..

**Executing the request**
1. Open the `Get Hosts` request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/envtoken/getHostsResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in dynatrace and it has the correct roles. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check that the prodPublicHostName & testPublicHostName environment variables have been set in postman.

![](./images/envtoken/getHostsVars.png)

Congratulations you have just collected information on your hosts using an API call. The same principals can be applied to all the data APIs.

# 5. Creating dynatrace auto tags

## Creating dynatrace auto tags background 

**What are dynatrace auto tags?**

In dynamic or large environments, manual tagging can be impractical. In such cases, it’s recommended that you use automated, rule-based tagging. Automatically applied tags behave just like the manually applied, except they’re applied automatically to new entities that match defined rules. Automatically applied tags can’t be removed manually from individual services, process groups, process group instances, applications, or hosts. Automatically applied tags are removed automatically once an entity no longer matches a defined rule.

**When would you create dynatrace auto tags?**

Managing huge amounts of information and organizing such large monitoring environments is a real challenge. To effectively cope with this challenge, Dynatrace supports tags and metadata. Tags and metadata enable you to organize your monitored environments in a meaningful way. Tags in Dynatrace are basically labels or markers while metadata are key/value pairs that are inherent to any monitored entity. Metadata are mainly used for defining extra information for entities while tags are used for organizing entities. You can also create tags based on metadata.

**How do you create dynatrace auto tags?**

Dynatrace auto tag rules can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create 2 auto tagging rules, one will leverage our host groups to identify the correct environment and the other will leverage the metadata on our host to identify the cost center.

## Creating dynatrace auto tags configuration

**Request configuration**

**ATTENTION:** There is currently no V2 autoTags API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint autoTags. This is different than the previous endpoints as this is now a config endpoint where we have previously used an entity endpoint. Config endpoints apply specifically to the specified environment.  

By making a request to this API endpoint we will create new auto tagging rules on our environment. In our example we will create 2 auto tagging rules. One will be called environment and the value will be the host group. This means in the future if we add another host group such as Dev it will be automatically be tagged with the correct value. The second rule will leverage the meta data we supplied when creating the hosts. We will create a tag called Cost Center and the value will be taken from the meta data parameter Cost_Center. The will allow us to pass the cost center to process groups and services that run on the host. This could be used later for cross charging.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create 2 distinct rules so the bodies will be different. Therefore the body only contains the postman environment variable \{\{reqBody\}\} which gets set by the pre-request script.

The following lists the parameters that will be set for each request

For the Environment tag will be set as 

Key | Value | Description
------------ | ------------- | -------------
name | Environment | This will be the name of our tag. Be sure to provide a meaningful name for each tag you create. 
type | PROCESS_GROUP | This is the entity we want our rule to be applied to, in our case we will choose process group.
enabled | true | This means we want the rule to active straight away. If you are setting something up in advance you may set this to false and enable it later.
valueFormat | \{HostGroup:Name\} | For our environment tag we want to set the value to the host group name, so we set this to the property HostGroup:Name.
propagationTypes | PROCESS_GROUP_TO_HOST & PROCESS_GROUP_TO_SERVICE | This tells the rule that we want to also push the tag to Host and Service level for matching process groups.
key | HOST_GROUP_NAME | We will set a condition to say only apply the tag if a host group exists

For the Cost Center tag will be set as 

Key | Value | Description
------------ | ------------- | -------------
name | Cost Center | This will be the name of our tag. Be sure to provide a meaningful name for each tag you create. 
type | PROCESS_GROUP | This is the entity we want our rule to be applied to, in our case we will choose process group.
enabled | true | This means we want the rule to active straight away. If you are setting something up in advance you may set this to false and enable it later.
valueFormat | \{HostGroup:Name\} | For our Cost Center tag we want to set the value to the host meta data property Cost_Center, so we set this to the property Host:Environment:Cost_Center.
propagationTypes | PROCESS_GROUP_TO_HOST & PROCESS_GROUP_TO_SERVICE | This tells the rule that we want to also push the tag to Host and Service level for matching process groups.
key | HOST_CUSTOM_METADATA, source: ENVIRONMENT, key: Cost_Center | We will set a condition to say only apply the tag if a cost center exists

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the new tag IDs and sets them as environment variable envTagID & ccTagID so we can use them later to modify the tag rules. The script also controls setting which rule we are running and throws an error if the request is executed again after both tags have been created.

**Executing the request**
1. Open the `Create New Auto Tag` request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createEnvTagResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write COnfig API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Execute the request again to create the Cost Center tag.
5. Check that the request received a `201 Created` response.

![](./images/envtoken/createCCTagResp.png)

    If you get an error follow the troubleshooting above.

6. Check that the envTagID & ccTagID environment variables have been set in postman

![](./images/envtoken/createTagVars.png)

7. View the results in your dynatrace environment.

![](./images/envtoken/tagsDT.png)

Congratulations you have just created 2 new auto tags via an API call. Now Let's leverage those tags to create management zones.

# 6. Creating dynatrace management zones

## Creating dynatrace management zones background 

**What are dynatrace management zones?**

Management zones are a powerful information-partitioning mechanism that simultaneously promote collaboration and the sharing of relevant team-specific data while also ensuring secure access controls.

Each customizable management zone comprises a set of monitored entities in your environment, be it hosts that share a common purpose, a specific application, a stagging environment, or services of a certain technology. Entities are readied and grouped for inclusion in management zones by the powerful Dynatrace tagging engine based on tags applied.

Management zones may overlap, just as team responsibilities often overlap. Users may be granted access to entire environments, a specific management zone, or a subset of related management zones. Each Dynatrace user sees an auto-filtered view comprised of only the management zones that they have permission to view. Users who have access to multiple management zones can focus on any particular management zone and filter all Dynatrace views so that they only see the metrics that are related to the entities of that particular management zone.

**When would you create dynatrace management zones?**

Particularly within large cloud environments, it’s critical that you have the ability to define fine-grained access to specific monitoring data based on the responsibilities of individual teams. Dynatrace management zones enable you to do this intuitively. Management zones allow you to define visibility and responsibility for monitoring data across all layers of your technology stack without reintroducing stack-related silos.

**How do you create dynatrace management zones?**

Dynatrace management zones can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will 2 management zones to separate our environments, one will be for the test environment and one for production.

## Creating dynatrace management zones configuration

**Request configuration**

**ATTENTION:** There is currently no V2 management zones API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint managementZones.  

By making a request to this API endpoint we will create new management zones rules on our environment.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create 2 distinct rules so the bodies will be updated with environment variables to set the correct values.

We will supply 3 rules for each management zone. The rules will cover services, processes, process groups and web applications to ensure all components are correctly mapped. For each of these levels we will perform the mapping based on the environment tag we created in the previous step. This means that future instances would be automatically mapped if setup correctly.

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct management zone  when requests are executed and updates the request body to be sent. This is because it is not possible to have a dynamic value for management zone names like we did for the environment tag value so we have to create a separate management zone rule for each one.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the new management zone IDs and sets them as environment variables prodMZID & testMZID so we can use them later to create alerting profiles. The script also controls setting which rule we are running and throws an error if the request is executed again after both management zones have been created.

**Executing the request**
1. Open the `Create Management Zones` request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createMZResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Execute the request again to create the second management zone.
5. Check that the request received a `201 Created` response.
    If you get an error follow the troubleshooting above.

6. Check that the prodMZID & testMZID environment variables have been set in postman

![](./images/envtoken/createMZVars.png)

7. Try out your new management zones in dynatrace. It is now much easier to find the correct host.

![](./images/envtoken/mzDT.png)

Congratulations you have just created 2 new management zones with via an API call. Now Let's set up some web applications so we can correctly identify our RUM data.

# 7. Creating dynatrace web applications

## Creating dynatrace web applications background 

Have a look at the applications in your environment. You should see just My web application \(if you don't see any make sure to set your Management Zone filter to All\). This is the default application and it can't be delete. Any RUM traffic that doesn't match an application detection rule will end up here. We are running effectively 4 applications whioch are easyTravel and easyTravel Angular in prod and test. We need to ensure we separate these.

![](./images/envtoken/mzDT.png)

**What are dynatrace web applications?**

Applications within Dynatrace are logical constructs onto which customer applications—web sites, mobile apps etc.—are mapped for monitoring with regard to traffic from real users. It is implied therefore that such customer applications have an end-user interface. Typical end-user interfaces include browser-based interfaces of web applications or web sites and iOS- or Android-based interfaces of mobile apps running on smartphones or tablets.

**When would you create dynatrace web applications?**

For all application types except for web applications, the definition of the Dynatrace application is generated when a customer creates the application and starts setting up the Dynatrace monitoring. These application types are not monitored in an automated way. For web applications, this definition is automatically generated. More specifically, if your web applications are running on systems where you can install Dynatrace OneAgent, Dynatrace will automatically inject the RUM JavaScript code into the HTML pages. Therefore, every monitoring environment obtains automatically a default application named “My web application”. All RUM data are assigned to this application by default. To provide better value from the solution you can create custom web applications to separate the RUM data.

**How do you create dynatrace web applications?**

Dynatrace  web applications can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create 4 web applications to separate our RUM data. Our ec2 instances each rum 2 applications, easyTravel and easyTravel Angular. We will create apps for each of these in both  test and production. My Web application is just a container for the traffic, in order for the correct traffic to be associated with the correct web application we will then need to provide application detection rules, this will be done in the next step.

## Creating dynatrace web applications configuration

**Request configuration**

**ATTENTION:** There is currently no V2 web application API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint applications/web.  

By making a request to this API endpoint we will create web applications on our environment.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create 4 distinct rules so the bodies will be updated with environment variables to set the correct values.

As part of the body we must supply the settings for the application. This is where you can control things like the thresholds and javaScript Framework Support. These settings can also be updated in existing applications with an API call later.

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct application name  when requests are executed and updates the request body to be sent.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the new application IDs and sets them as environment variables so we can use them later to create application detection rules. The script also controls setting which rule we are running and throws an error if the request is executed again after all four applications have been created.

**Executing the request**
1. Open the `Create New Web Application` request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createWebAppResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Execute the request 3 more times to create the other web applications.
5. Check that the request received a `201 Created` response.
    If you get an error follow the troubleshooting above.
6. Check that the etAngProdAppID, etAngtestAppID, etProdAppID & etTestAppID environment variables have been set in postman

![](./images/envtoken/createWebAppVars.png)

7. Check that the web applications have been created in dynatrace. Please note no data will be mapped to these yet as all we have done is create the applications. Application detection rules are needed to map the traffic to the correct application and we will create these in the next step. You can check they have been created by navigating to Settings > Monitoring > Monitoring Overview > Applications

![](./images/envtoken/webAppsDT.png)

Congratulations you have just created 4 new web applications via an API call. Now let's create the app detection rules to correctly map our RUM traffic to them.

# 8. Creating dynatrace application detection rules

## Creating dynatrace application detection rules background 

**What are dynatrace application detection rules?**

Once OneAgent is installed on a host, it monitors all applications running on the host. All monitoring data is encapsulated into a sample placeholder application called My web application. The reason we offer this placeholder application is to allow for more flexibility, as we leave it up to you how your application should be organized. There are cases, for example, where you may want to merge multiple domains into a single application. This is typically the case for sites that are offered in multiple languages, for example www.myshop.de and www.myshop.at. Application detection rules allow you to control this.

**When would you create dynatrace application detection rules?**

You would create application detection rules when onboarding a new RUM application that is automatically instrumented via the OneAgent. Separating applications by top-level domain is always a good idea, as Dynatrace can’t correlate user actions across top-level domains to user sessions anyway. This correlation is done via a cookie and therefore only works if the cookie can be set on the same top-level domain. For example, user actions for www.dynatrace.com and blog.dynatrace.com can be captured in one application as the cookie can be set to dynatrace.com, but traffic for www.dynatrace.com and www.internal-dynatrace.com can’t be captured in one user session. You will still be able to separate user actions based on the domain, but user sessions will never have actions from both domains.

**How do you create dynatrace application detection rules?**

Dynatrace application detection rules can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create rules to correctly map our RUM traffic to the 4 web applications we created in the previous step.

## Creating dynatrace application detection rules configuration

**Request configuration**

**ATTENTION:** There is currently no V2 application detection rules API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint applicationDetectionRules.  

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create 4 distinct rules so the bodies will be updated with environment variables to set the correct values.

The two primary things we are setting here is the app IDs from the previous step and the rules to match the traffic. In our case we will match the traffic using the public DNS name of the host \(which was captured when we executed the Get Hosts request\) and the port number the app is running on. 

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct application ID and pattern rule when requests are executed and updates the request body to be sent.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script controls setting which rule we are running and throws an error if the request is executed again after all four application detection rules have been created.

**Executing the request**
1. Open the `Create New App Detection Rules` request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createAppDetResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Execute the request 3 more times to create the other application detection rules.
5. Check that the request received a `201 Created` response.
    If you get an error follow the troubleshooting above.
6. Only new traffic will be mapped to our new applications. Wait a few minutes then check your applications page and you should see traffic in all four of you new applications. 

![](./images/envtoken/createAppDetDT.png)

Congratulations you have just created 4 new application detection rules via an API call. 

# 9. Modifying dynatrace auto tags

## Modifying dynatrace auto tags background 

**Why are we modifying the dynatrace auto tags?**

In your dynatrace environment have a look at your applications. Have they been assigned the environment tag? Do they appear in your management zones?

In real-life scenarios you will find that you often need multiple tagging rules for them to function as you would like.


## Modifying dynatrace auto tags configuration

**Request configuration**

**ATTENTION:** There is currently no V2 autoTags API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This leverages the same config v1 API endpoint autoTags that we used to create the tags but this time it will be a PUT request. You will notice in the URL we are now specifying an environment variable in the parameter \{\{envTagID\}\}, this is the ID of the environment tag in dynatrace that was captured when it was created during the `Create New Auto Tag` request. To modify an existing auto tag you must provide the relevant ID. This could also be gathered either in the UI or by calling the autoTags API endpoint with a GET request.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we will supply the same process group rule for the tag

The body will be the same as the previous body but this time with 3 rules instead of one. The process group rule will be the same, but we will also add a rule for services and web applications. We need to add the services level rule to cover services that are not assigned to a process group, for example volatile services and a web application rule to cover our web apps as they can't use the host group property. For the web applications we will use a string match on the application names as when we created them we added the environment name to their names.

**Executing the request**
1. Open the Modify New Auto Tag request.
2. Click on `Send` to execute the request.
3. Check that the request received a `204 No Content` response.

![](./images/envtoken/modifyTagResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Inside your dynatrace UI check that the web apps have now have the environment tag added to them.

![](./images/envtoken/modifyTagApps.png)

Congratulations you have just modified your first dynatrace configuration via an API call. While we only modified our environment tag, the same issue could be said for the cost center tag. In the future feel free to try modifying that one yourself.

# 10. Creating dynatrace request attributes rules

## Creating dynatrace request attributes background 

**What are dynatrace request attributes?**

Request attributes are essentially key/value pairs that are associated with a particular service request. For example, if you have a travel website that tracks the destinations of each of your customers’ bookings, you can set up a destination attribute for each service request. The specific value of the destination attribute of each request is populated for you automatically on all calls that include a destination attribute

**When would you create dynatrace request attributes?**

There are many use cases for request attributes. You can also configure custom request attributes that you can use to improve filtering and analysis of problematic web requests. In our case we will use a request attribute to extract our user's loyalty status. This means we will be able to filter our data and report on users of different loyalty statuses. In the future you could even take this further and integrate the information with your CRM to automatically reach out to your top-level users if they experience issues.

**How do you create dynatrace request attributes?**

Dynatrace request attributes can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create the loyalty status request attribute via an API call.

## Creating dynatrace request attributes configuration

**Request configuration**

**ATTENTION:** There is currently no V2 request attributes API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Before you complete this request please ensure you have enabled real time updates to java services as per the prerequisites.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint service/requestAttributes.  

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to supply 2 rules for the Loyalty Status request attribute. The first rule will look for the method name `checkLoyaltyStatus` in the `com.dynatrace.easytravel.business.webservice.BookingService` class and the second will look for the method `getLoyaltyStatus` in  the `com.dynatrace.easytravel.business.webservice.AuthenticationService`. 

**Executing the request**
1. Open the Create Request Attribute request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createReqAttResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.


Congratulations you have just created a new request attribute via an API call. 

# 11. Creating dynatrace load test request attributes rules

## Creating dynatrace load test request attributes background 

**What are dynatrace load test request attributes?**

These are the same as the request attribute created before but this time we want to create some request attributes so we can integrate dynatrace with our chosen load test tool. In this case we will use postman runner to generate the load later on, but the principal is the same for any load test tool.

**When would you create dynatrace load test request attributes?**

While executing a load test from your load testing tool of choice (JMeter, Neotys, LoadRunner, etc) each simulated HTTP request can be tagged with additional HTTP headers that contain test-transaction information (for example, script name, test step name, and virtual user ID). Dynatrace can analyze incoming HTTP headers and extract such contextual information from the header values and tag the captured requests with request attributes. Request attributes enable you to filter your monitoring data based on defined tags.

**How do you create dynatrace load test request attributes?**

Dynatrace request attributes can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create the request attribute via an API call.

Later on when we run our load test we have added the header `x-dynatrace-test` with the following set of key/value pairs for the header:

1. TSN - Test Step Name is a logical test step within your load testing script (for example, Login or Add to cart.
2. LSN - Load Script Name is the name of the load testing script. This groups a set of test steps that make up a multi-step transaction (for example, an online purchase).
3. LTN - The Load Test Name uniquely identifies a test execution (for example, 6h Load Test – June 25)

In real-life you may have additional key/value pairs to provide additional contextual information.

## Creating dynatrace load test request attributes configuration

**Request configuration**

**ATTENTION:** There is currently no V2 request attributes API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint service/requestAttributes.  

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create 3 distinct rules so the bodies will be different. The pre-request script will be used to set the correct value of the variable \{\{curRA\}\}. The body will be the same other than this variable as all the request attributes will be extracted from the same `x-dynatrace-test` header but we perform an extraction on the key name to find the correct one.

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct request attribute when requests are executed and updates the request body to be sent.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script controls setting which rule we are running and throws an error if the request is executed again after all three load test request attribute rules have been created.

**Executing the request**
1. Open the Create Load Test Request Attribute request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createLTReqAttResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.


Congratulations you have just created 3 new request attributes via an API call. 

# 12. Creating dynatrace calculated service metrics rules

## Creating dynatrace calculated service metrics background 

**What are dynatrace calculated service metrics?**

Dynatrace automatically captures important metrics for services with no configuration required. Additionally you might need additional business or technical metrics that are specific to your application. These metrics can be calculated and derived based on a wide variety of available data within the captured PurePath. This allows you to further customize key performance metrics for which alerts should be generated and helps you keeping an eye on them by pinning them to your dashboards.

**When would you create dynatrace calculated service metrics?**

There are many use cases for calculated service metrics. For example, a ride-hailing company might want to keep track of the number of active drivers per region, or the average ride distance, and be alerted on significant anomalies.

In our example we will leverage our previously created loyalty status request attribute to calculate the number of bookings for each loyalty status.

**How do you create dynatrace calculated service metrics?**

Dynatrace calculated service metrics can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create the bookings by loyalty status calculated metric via an API call.

## Creating dynatrace calculated service metrics configuration

**Request configuration**

**ATTENTION:** There is currently no V2 calculated service metric API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Before you complete this request please ensure you have enabled real time updates to java services as per the prerequisites.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint service/requestAttributes.  

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create a single calculated service metric for bookings by loyalty status. We will specify a unique metric key under tsmMetricKey, that we want a request count and the conditions that request name is storeBooking and the request attribute loyalty status exists.

**Executing the request**
1. Open the Create Calculated Service Metric request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createCSMResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.


Congratulations you have just created a new calculated service metric via an API call. 

Wait a few minutes then create a custom chart to view the results.

# 13. Modifying dynatrace web applications

## Modifying dynatrace web applications background 

**Why are we modifying the dynatrace web applications?**

In real-life environments there will be times when you need to change configurations, such as application updates, SLA changes etc. In your dynatrace environment have a look at your easyTravel Angular - Production application. You should notice that there are some javascript frameworks detected that are not enabled. We also have no conversion goals defined.

We are going to enable the angular javascript framework and set a conversion goal.

## Modifying dynatrace web applications configuration

**Request configuration**

**ATTENTION:** There is currently no V2 web application API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This leverages the same config v1 API endpoint applications\/web that we used to create the application but this time it will be a PUT request. You will notice in the URL we are now specifying an environment variable in the parameter \{\{etAngProdAppID\}\}, this is the ID of the web application in dynatrace that was captured when it was created during the `Create New Wen Application` request.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case as this is modifying an existing application the body will be similar to the body we used to create it, but this time we will set angular to true and we supply the conversion goal settings which you will see at the bottom. In real-life you could also poll the get a application API first to gather the current config which could then be used to create the new body.

**Executing the request**
1. Open the Modify Web Application request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/modifyWebAppResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

Congratulations you have just modified your web applications via an API call.

# 14. Creating dynatrace alerting profiles

## Creating dynatrace alerting profiles background 

**What are dynatrace alerting profiles?**

Alerting profiles control the delivery of problem notifications across your organization's alerting channels based on considering predefined filters that are based on problem severity, problem duration, custom events, and tags. Alerting profiles allow you to control exactly which conditions result in problem notifications and which don't.

**When would you create dynatrace alerting profiles?**

There are many use cases for alerting profiles. For example, you might use an alerting profile to filter for issues that affect production components only and use that in your problem notification to your ticketing system.

In our example we will create 2 alerting profiles for our previously created management zones, Production and Test. This way we can later filter our problem notifications for each management zone.

**How do you create dynatrace alerting profiles?**

Dynatrace alerting profiles can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create the alerting profiles via an API call.

## Creating dynatrace alerting profiles configuration

**Request configuration**

**ATTENTION:** There is currently no V2 alerting profiles API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint alertingProfiles.  

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create 2 alerting profiles, one for Production and one for Test. In order to simplify this we will use our management zones as the filer. This means in the future, give the way we have set up this environment, if we roll out further hosts with the correct host group they will automatically be tagged and assigned to the correct management zone. This will mean they will also be automatically be applied to the correct alerting profile. In real-life you could also use alerting profiles to add additional filers.

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct management zone when requests are executed and updates the request body to be sent.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script controls setting which rule we are running and throws an error if the request is executed again after both alerting profiles have been created.

**Executing the request**
1. Open the Create Alerting Profile request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createAPResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.


Congratulations you have just created 2 alerting profiles via an API call. 

# 15. Creating dynatrace email notifications

## Creating dynatrace email notifications background 

**What are dynatrace email notifications?**

Dynatrace email notifications send an email to a specified address or addresses when problems are opened.

**When would you create dynatrace email notifications?**

Dynatrace offers several out-of-the-box integrations that automatically push Dynatrace problem notifications to your third-party messaging or incident-management systems. If however your third-party system isn't supported with an out-of-the-box integration, you can easily set up email integration.

We will set up to integrations, email and a custom webhook. We will use our previously created alerting profiles to send Production problems to email and Test problems to our webhook \(which we will setup in the next request\).

**How do you create dynatrace email notifications?**

Dynatrace email notifications can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create the email notification via an API call.

## Creating dynatrace email notifications configuration

**Request configuration**

**ATTENTION:** There is currently no V2 notifications API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the config v1 API endpoint notifications.  

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create an email notification that will go to the address you configured in the prerequisites. The type will be EMAIL and we will specify the address using the environment variable \{\{email\}\} and the production alerting profile filter that we created previously using the environment variable \{\{prodAPID\}\}. You can also control the subject line and body. Our subject line will contain the state of the problem, the problem ID and the impacted entity. This way we will know what is going on from just the email subject. For the body we will simply include the problem details in HTML format to give us more information.

**Executing the request**
1. Open the Create Email Notifications request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createEmailResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

Congratulations you have just created an email notification via an API call. Now we will set up a custom webhook notification.

# 16. Creating dynatrace custom webhook notifications

## Creating dynatrace custom webhook notifications background 

**What are dynatrace custom webhook notifications?**

Dynatrace webhook notifications send an HTTP POST request to a specified url when problems are opened. The payload message of the HTTP POST request is completely customizable. By default, requests use valid JSON syntax except that you define a different HTTP content type header. In case you define a different HTTP content type header, Dynatrace will skip the JSON validation. In such cases, Dynatrace doesn't escape the payload based on JSON syntax. Information placeholders, such as {ProblemTitle} and {State}, are used to fill the custom JSON with the dynamic information of each detected problem.

**When would you create dynatrace custom webhook notifications?**

Dynatrace offers several out-of-the-box integrations that automatically push problem notifications to third-party incident-management and ChatOps systems. If however your third-party system isn't supported with an out-of-the-box integration, you can easily set up a customizable webhook integration. Using this approach, whenever Dynatrace detects a problem in your environment that affects real users, a webhook triggers an HTTP POST request to a target URL that you specify. 

We have already create an email notification for production issues so now we will create a webhook notification for our Test environment.

**How do you create dynatrace custom webhook notifications?**

Dynatrace webhook notifications can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create the webhook notification via an API call.

## Creating dynatrace custom webhook notifications configuration

**Request configuration**

**ATTENTION:** There is currently no V2 notifications API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the same config v1 API endpoint notifications. The type of the notification is controlled by the post body.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server. In our case we want to create a webhook notification that will go to the webhook.site you configured in the prerequisites. This time the type will be WEBHOOK and we will specify the url using the environment variable \{\{webhook\}\} and the test alerting profile filter that we created previously using the environment variable \{\{testAPID\}\}. This time the information we want to supply is provided in the payload.

**Executing the request**
1. Open the Create Webhook Notifications request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createWHResp.png)

    If you get a could not send request error check the value of your dtURL environment variable. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in your environment and it has the Write Config API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

Congratulations you have just created an email notification via an API call. Now we will set up a custom webhook notification.

# 17. Starting an easyTravel problem pattern

The easyTravel Business Backend process provides REST interfaces for controlling plugins, i.e. listing available and enabled plugins and enabling/disabling plugins. As this is not a dynatrace API call and is purely to start a problem patter in easyTravel we will not cover this in as much detail. For more information on it please see the [easytravel documentation](https://community.dynatrace.com/community/display/DL/Available+easyTravel+Problem+Patterns). 

In our case we are going to enable the plugin SlowApacheWebserver on our test host which slows down Apache Webserver. If you wish to disable it again later simply change the parameter enabled from true to false.

**Executing the request**
1. Open the Start ET problem request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/ami/getstartETPResp.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

5. Wait a few minutes then you should receive a problem notification you your webhook.site

![](./images/ami/getstartETPN.png)

Now that we have the latest Ubuntu image AMI we can launch our ec2 instances.

