---
title: AWS CodePipeline - Integrating with TeamCity
date: 2017-11-05 21:53:38
tags:
---


AWS CodePipeline is an excellent tool for orchestrating your deployments in the cloud. It provides out of the box integration with  CI Providers such as CodeBuild, Jenkins and Solano CI. Other providers can be configured as well, like TeamCity, and that's the aim of this post.

In order to understand the extensibility point in CodePipeline that allows to plugin a custom action like a TeamCity Build, read http://docs.aws.amazon.com/codepipeline/latest/userguide/actions-create-custom-action.html which explains in depth custom action types in CodePipeline.

We will be setting up a deployment pipeline to automate releases of a simple nodejs app hosted in GitHub. To provision the Pipeline, we will use CloudFormation, which means you can provision it in an automated fashion. 

So, let's start by creating our project folder, let's call it `node-app-devops`. Next, create a directory inside the project called `templates`, which is where your cloud formation template will be, let's name it `deployment-pipeline.yaml`.


Open `deployment-pipeline.yaml` in an editor of your preference and let's start by adding a **Description** to your template, and the **Parameters**.

```
Description: >
    This template will provision the  CodePipeline for the App.
    (Pipeline) GitHub -> TeamCity

Parameters:
   
    GitHubOwner:
        Description: Repository Owner
        Type: String
    
    GitHubRepo:
        Description: Repository Name
        Type: String
   
    GitHubBranch:
        Description: Repository Branch
        Type: String
   
    GitHubOAuthToken:
        Description: Access Token Generated via GitHub (https://github.com/settings/tokens)
        Type: String
   
    TeamCityServerUrl:
        Description: Url for TeamCity. Expected URL format, http[s]://host[:port]
        Type: String
   
    TeamCityBuildId:
        Description: Build Id for TeamCity Project
        Type: String
   
    TeamCityActionId:
        Description: Action Id for Codepipeline integration with TeamCity
        Type: String
```

`GitHubOwner` would be simply your GitHub username, `GitHubRepo` the name of the repository and `GitHubBranch` is the branch where CodePipeline will fetch from. `GitHubOAuthToken` can be created if you don't have one already, by going to https://github.com/settings/tokens.

So far, it should be looking like this:

![Description And Parameters](https://res.cloudinary.com/danielcondemarin/image/upload/v1509871274/AWS-CodePipeline-Integrating-with-TeamCity/description_and_parameters.png)


Next, let's work on the `Resources` section, starting with the IAM Role used by the Pipeline. The role will provide access to an S3 Bucket, which will store the artifacts.

```
Resources:
   
    CodePipelineServiceRole:
        Type: AWS::IAM::Role
        Properties:
            RoleName: !Sub 'cpsr-${AWS::StackName}'
            Path: /
            AssumeRolePolicyDocument: 
                Statement:
                    -
                        Effect: Allow
                        Principal:
                            Service:
                                - codepipeline.amazonaws.com
                        Action:
                            - sts:AssumeRole
            Policies:
                - 
                    PolicyName: !Sub 'cpsrp-${AWS::StackName}'
                    PolicyDocument:
                        Statement:
                            - Resource:
                                - !Sub 'arn:aws:s3:::${ArtifactBucket}/*'
                              Effect: Allow
                              Action: 
                                - s3:PutObject
                                - s3:GetObject
    
    ArtifactBucket:
        Type: AWS::S3::Bucket
        DeletionPolicy: Retain

```

Note, how in **RoleName** I'm using `cpsr-${AWS::StackName}`, as convention **cpsr** stands for the initials of the Resource being provisioned, and we append the Pseudo Parameter `AWS::StackName` to have a more unique name which can also be easily associated to the Stack.
In the `Policies` there will be one that provides the S3 access needed by the Pipeline.`ArtifactBucket` is simply the S3 bucket that will be used as intermediate storage between TeamCity and CodePipeline.

Next, let's do the actual integration with TeamCity on our pipeline. For that, we need a [Custom Action Type](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-codepipeline-customactiontype.htm):

```

    TeamCityBuildActionType:
        Type: AWS::CodePipeline::CustomActionType
        Properties:
            Version: 1
            Category: Build
            Provider: TeamCity
            Settings:
                EntityUrlTemplate: '{Config:TeamCityServerURL}/viewType.html?buildTypeId={Config:BuildConfigurationID}'
                ExecutionUrlTemplate: '{Config:TeamCityServerURL}/viewLog.html?buildId={ExternalExecutionId}&tab=buildResultsDiv'
            ConfigurationProperties:
                - Name: TeamCityServerURL
                  Description: The expected URL format is http:[s]://host[:port]
                  Required: true
                  Key: true
                  Secret: false
                  Queryable: false
                  Type: String
                - Name: BuildConfigurationID
                  Description: TeamCity configuration external Id
                  Required: true
                  Key: true
                  Secret: false
                  Queryable: false
                  Type: String
                - Name: ActionID
                  Description: > 
                      Must be unique, match the corresponding field in TeamCity build trigger settings and satisfy regular expression pattern: [a-zA-Z0-9_-]+] and have length <= 20
                  Required: true
                  Key: true
                  Secret: false
                  Queryable: true
                  Type: String
            InputArtifactDetails:
                MaximumCount: 5
                MinimumCount: 0
            OutputArtifactDetails:
                MaximumCount: 5
                MinimumCount: 0

```

Note that ActionID is the value that will link TeamCity together with CodePipeline, but more on that later.

Lastly, let's bring all the pieces together and implement the actual CodePipeline:

```

    Pipeline:
        Type: AWS::CodePipeline::Pipeline
        Properties:
            RoleArn: !GetAtt CodePipelineServiceRole.Arn
            ArtifactStore:
                Type: S3
                Location: !Ref ArtifactBucket
            Stages:
                - Name: Source
                  Actions:
                      - Name: GitHubSource
                        ActionTypeId: 
                            Category: Source
                            Owner: ThirdParty
                            Version: 1
                            Provider: GitHub
                        Configuration:
                            Owner: !Ref GitHubOwner
                            Repo: !Ref GitHubRepo
                            Branch: !Ref GitHubBranch
                            OAuthToken: !Ref GitHubOAuthToken
                        OutputArtifacts:
                            - Name: AppRepo
                - Name: Build
                  Actions:
                      - Name: TeamCityBuild
                        ActionTypeId:
                            Category: Build
                            Owner: Custom
                            Version: 1
                            Provider: TeamCity
                        Configuration:
                            TeamCityServerURL: !Ref TeamCityServerUrl
                            BuildConfigurationID: !Ref TeamCityBuildId
                            ActionID: !Ref TeamCityActionId
                        InputArtifacts:
                            - Name: AppRepo
                        OutputArtifacts:
                            - Name: BuildOutput

```

I know what you're thinking, there is a missing piece, which is the deployment stage right? :) Well the purpose of the post is to show how to integrate with TeamCity your CodePipeline, so we'll keep it simple, and maybe have another post later on how to integrate with other services for deploying.

Let's now create the BuildConfiguration in TeamCity for the app, but first, we need to install a plugin which will be responsible for pulling any build jobs provided by the Pipeline. The plugin can be found [here](https://confluence.jetbrains.com/display/TW/AWS+CodePipeline+Plugin). If you've never installed a plugin before on TeamCity, just go to **Administration > Plugins List** and click on **Upload plugin zip**. You can get the plugin zip from the link above. You might need to restart the TeamCity server after uploading the plugin. Should look this after installing it:

![TeamCity Plugins](https://res.cloudinary.com/danielcondemarin/image/upload/v1509908370/AWS-CodePipeline-Integrating-with-TeamCity/teamcityplugins.png)

Note how I've got a plugin for better nodejs integration with TeamCity, that will come handy when we configure our build. You can find it [here](https://github.com/jonnyzzz/TeamCity.Node), install it like you did for the previous one.

Create a new Project in TeamCity, and a build configuration, skip the VCS setup, since our repository will come from CodePipeline instead of a version control provider. I named my project `Demo App`, and the build configuration `DemoAppBuild`. First, we need to configure a Trigger, which will execute the build configuration whenever the Pipeline needs it to. If you've installed correctly the CodePipeline plugin for TeamCity, you should be able to configure the trigger like this:
![CodePipeline Trigger](https://res.cloudinary.com/danielcondemarin/image/upload/v1509909920/AWS-CodePipeline-Integrating-with-TeamCity/codepipelinetrigger.png)
Use your own AWS Access Keys. Also, in ActionID just put anything meaningful, this will be the value that will tie together your build configuration with the Pipeline in AWS.

Now let's configure the Build Steps. I have 5 build steps:

![Build Steps](https://res.cloudinary.com/danielcondemarin/image/upload/v1509910295/AWS-CodePipeline-Integrating-with-TeamCity/buildsteps.png).

Steps 1, 3 and 4 can be configured easily by using the johnnyzzz plugin mentioned above, so I won't go in detail for these. Now, let's have a look at step `2. Copy Repository To Working Directory`. This is a command line script which will pickup the repository dropped by CodePipeline and copy it to the build working directory:

```
#!/bin/bash

# vars
repo_input_zip="%codepipeline.artifact.input.folder%/AppRepo.zip"

# unzip repo to working directory
unzip $repo_input_zip -d %teamcity.build.workingDir%
```

The variable `%codepipeline.artifact.input.folder%` is automatically setup by the plugin, and points to the directory where the repository will be dropped.

Let's have a look now at the last step: `5. Copy Repository To CodePipeline Output Directory`:

```
#!/bin/bash

# vars
repo_output_zip="%codepipeline.artifact.output.folder%/BuildOutput.zip"

# zip & copy repo to output directory for codepipeline
zip -r $repo_output_zip %teamcity.build.workingDir%
```

This step will basically copy the build artifact to the pickup directory for CodePipeline to copy from.

It is demo time now, so let's go ahead now and launch our stack:

[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=AppPipeline&templateURL=https://s3-eu-west-1.amazonaws.com/devpanda-bucket/node-app-devops/cf-templates/deployment-pipeline.yaml)

After following the Launch Stack link, you should get a screen like this:

![Stack Launch](https://res.cloudinary.com/danielcondemarin/image/upload/v1509914716/AWS-CodePipeline-Integrating-with-TeamCity/cf-launch-parameters.png)

Make sure you replace `GitHubOAuthToken` with your own and the `TeamCityServerUrl` as well. The app node-demo-app is hosted on my account [here](https://github.com/danielcondemarin/node-demo-app)
You should have access to it with your token, if not, just fork it and use  your own username and repository. Now follow the Stack creation, and after completion it should look like:

![Stack Created](https://res.cloudinary.com/danielcondemarin/image/upload/v1509915058/AWS-CodePipeline-Integrating-with-TeamCity/stackcompleted.png)


After the Stack is created, it will automatically trigger the CodePipeline which should all work as expected. 

![Pipeline Success](https://res.cloudinary.com/danielcondemarin/image/upload/v1509915396/AWS-CodePipeline-Integrating-with-TeamCity/codepipelinesuccess.png)

That is the end of this post, hope you enjoyed it, and is of good use in your devops toolbelt ;). All the sources for the demo can be found on the repo : https://github.com/danielcondemarin/node-app-devops.
