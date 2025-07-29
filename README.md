# CI/CD Pipeline with Canary Deployment on Elastic Beanstalk

This project implements a fully automated CI/CD pipeline that deploys a PHP web application to AWS Elastic Beanstalk with **canary deployment** using **Rolling with Additional Batch** strategy. The pipeline leverages **AWS CodePipeline**, **CodeBuild**, **S3**, and **CloudFormation** for infrastructure automation.

---

##  Project Goals

   - Set up GitHub for version control.
   - Create CodePipeline to automate the build and deployment phases.
   - Deployed app to Elastic Beanstalk with environment configurations.
   - Enabled CloudWatch monitoring for deployment health.
   - Configured canary deployments to limit rollout and observe system response.

---

## Technologies Used

- **AWS CloudFormation** – Infrastructure as Code
- **AWS Elastic Beanstalk** – App hosting with rolling deployments
- **AWS CodePipeline** – CI/CD orchestration
- **AWS CodeBuild** – Build logic for infra and app
- **Amazon S3** – Artifact storage
- **GitHub** – Source control for both infrastructure and application

---

##  Repository Structure

### 1. Infrastructure Repository
This repository contains CloudFormation templates for provisioning all AWS resources required to support the CI/CD pipeline and deployment.

#### Key Features:
- Creates Elastic Beanstalk application and environment.
- Provisions S3 bucket to store application artifacts.
- Manages environment configuration such as canary deployment settings.

#### Notable Files:
```
iac-infras-setup/
├── artifact-bucket.yaml      # S3 artifact bucket for application files
├── eb.yaml                   # Elastic Beanstalk app + env with rolling/canary deployment
├── master-stack.yaml         # Master template deploying nested stacks
├── buildspec.yml             # Builds and deploys infrastructure using CloudFormation
```
![GitHub](/elasticbean/Screenshot6.png)
![GitHub](/elasticbean/Screenshot5.png)

**FULL CODE**
```
AWSTemplateFormatVersion: '2010-09-09'
Description: Elastic Beanstalk app with environment

Resources:
  EBApplication:
    Type: AWS::ElasticBeanstalk::Application
    Properties:
      ApplicationName: MyApp
      Description: My Canary Deploy App

  EBApplicationVersion:
    Type: AWS::ElasticBeanstalk::ApplicationVersion
    Properties:
      ApplicationName: !Ref EBApplication
      Description: Initial version
      SourceBundle:
        S3Bucket: my-artifacts-azamcloud
        S3Key: app.zip

  EBEnvironment:
    Type: AWS::ElasticBeanstalk::Environment
    Properties:
      EnvironmentName: MyApp-env1
      ApplicationName: !Ref EBApplication
      SolutionStackName: "64bit Amazon Linux 2023 v4.7.1 running PHP 8.2"
      OptionSettings:
        - Namespace: aws:elasticbeanstalk:command
          OptionName: DeploymentPolicy
          Value: RollingWithAdditionalBatch
        - Namespace: aws:elasticbeanstalk:command
          OptionName: BatchSizeType
          Value: Percentage
        - Namespace: aws:elasticbeanstalk:command
          OptionName: BatchSize
          Value: 50
        - Namespace: aws:elasticbeanstalk:command
          OptionName: PauseTime
          Value: PT5M
        - Namespace: aws:autoscaling:launchconfiguration
          OptionName: IamInstanceProfile
          Value: aws-elasticbeanstalk-ec2-role
       
      VersionLabel: !Ref EBApplicationVersion
      Tier:
        Name: WebServer
        Type: Standard
```
![GitHub](/elasticbean/Screenshot7.png)
![GitHub](/elasticbean/Screenshot8.png)


---

### 2. Application Source Repository
This repository contains the PHP web application and its build logic.

#### Key Features:
- Builds application artifacts from HTML, JS, and CSS.
- Uploads application zip files to S3 bucket provisioned by the infrastructure stack.
- Triggers deployment on Elastic Beanstalk using AWS CodePipeline.

#### Notable Files:
```
eb-cicd/
├── index.html
├── script.js
├── styles.css
├── buildspec.yml         # Builds artifact, uploads to S3
```

![GitHub](/elasticbean/Screenshot2.png)
![GitHub](/elasticbean/Screenshot3.png)
![GitHub](/elasticbean/Screenshot4.png)
![GitHub](/elasticbean/Screenshot10.png)

---

##  CI/CD Pipeline Overview

### Pipeline Flow:

```
GitHub (Source Repos)
     │
     ├──> (Infrastructure)
     │       └──> CodeBuild (Deploys S3 Artifact bucket template stack)
     │
     ├──> (Application)
     |        └──> CodeBuild (Builds and uploads app artifact to S3 Artifact Bucket)
     |
     └──> (Infrastructure)     
              └──> CodeBuild (Deploys Elasticbean Template stack with application url pointing to Artifact bucket and objects)
                     ↓
              Elastic Beanstalk (Canary Deployment)
```


##  How It Works

1. **Infrastructure Build(Artifact bucket)**: Triggeres via changes to the infrastructure repo. Deploys or updates AWS resources(S3) using CloudFormation through CodeBuild.

![howitworks](/elasticbean/Screenshot9.png)
   
3. **App Build & Upload**: Triggeres on changes to the application repo. The buildspec script:
   - Zips relevant files.
   - Uploads to S3(Artifact Bucket).

  ![howitworks](/elasticbean/Screenshot12.png)
  ![howitworks](/elasticbean/Screenshot13.png)
  ![howitworks](/elasticbean/Screenshot11.png)
  

4. **Infrastructure Build(ElasticBeanstalk)**:  Triggeres via changes to the infrastructure repo. Deploys or updates AWS resources(ElasticBeanstalk) using CloudFormation through CodeBuild.

![howitworks](/elasticbean/Screenshot14.png)
![howitworks](/elasticbean/Screenshot16.png)
![howitworks](/elasticbean/Screenshot17.png)


   
2. **Canary Deployment**: Elastic Beanstalk is configured to replace 50% of instances per batch.
![howitworks](/elasticbean/Screenshot15.png)
---

## Additional Setup 
  Configured Codepipline with the Source ──> GitHub(Application repo), Build Stage(Application CodeBuild),Deploy Stage(ElasticBeanstalk)

  This directly updates the ElasticBeanstalk application with changes made to the source repo(trigger), using the build artifact

![additional](/elasticbean/Screenshot21.png)
![additional](/elasticbean/Screenshot20.png)
![additional](/elasticbean/Screenshot18.png)
![additional](/elasticbean/Screenshot19.png)

---
##  Monitoring & Logs

- **Elastic Beanstalk Console**: View deployment progress and logs.
- **CloudWatch Logs**: Monitor CodeBuild and environment logs.
- **CodePipeline Console**: Visual status of each pipeline stage.

---

