# Continuous Integration with CodeBuild

## Introducing this Project!

In this project, I set up CodeBuild to automate the build process of my CI/CD pipeline.

### Services Used

- IAM
- EC2
- CodeArtifact
- CodeBuild
- S3

### Concepts Learnt

- Creating CodeBuild projects
- Writing buildspec.yml file
- Connecting GitHub with AWS
- Starting builds and troubleshooting errors

### Architecture Diagram

![Image](https://github.com/sumeet15n/CI-with-CodeBuild/blob/master/Screenshots/SS0.png)

---

## Project Guide

### Step 1: Set up a web app in the cloud

Please refer to [my earlier project](https://github.com/sumeet15n/set-up-a-web-app-in-the-cloud) for this step.

### Step 2: Connect the web app to GitHub

Please refer to [my earlier project](https://github.com/sumeet15n/connect-a-GitHub-repo-with-AWS) for this step.

### Step 3: Create a CodeArtifact repository

Please refer to [my earlier project](https://github.com/sumeet15n/secure-packages-with-CodeArtifact) for this step.

### Step 4: Create a CodeBuild project

First, I created an S3 bucket with name 'nextwork-devops-cicd-sumeet' intended to be the place where CodeBuild will store the build file.

I then navigated to CodeBuild and created a CodeBuild project, with 'nextwork-devops-cicd' as the name and 'GitHub' as the source provider. 

I created a connection between my AWS account and GitHub account using 'AWS Connector for GitHub'. A screenshot showing the successfully established connection is provided below.

![Image](https://github.com/sumeet15n/CI-with-CodeBuild/blob/master/Screenshots/SS1.png)

Once my connection was established, I was able to select my GitHub repository as the 'Repository' and continue with the CodeBuild project creation process.

I de-selected the webhook option, chose on-demand as the provisioning model, chose managed image as the environment image, chose EC2 as the compute type, chose Amazon Linux as the operating system, selected standard as the runtime, chose aws/codebuild/amazonlinux-x86_64-standard:corretto8 as the image, and chose new service role as the service role.

In the Buildspec section, I selected 'Use a buildspec file' and left the name as 'buildspec.yml'.

In the Artifacts section, I selected Amazon S3 as the type, selected my S3 bucket named 'nextwork-devops-cicd-sumeet', entered 'nextwork-devops-cicd-artifact' as the name of the artifact, and selected zip as the artifact packaging option.

Lastly, I enabled CloudWatch logs and completed the CodeBuild project creation.

### Step 5: Run the build and troubleshoot failures

I navigated to my newly created CodeBuild project 'nextwork-devops-cicd' and clicked on 'Start build'. However, the status changed from in progress to failed.

I navigated to the Phase details tab and noticed that the DOWNLOAD_SOURCE phase had failed with the error message 'YAML_FILE_ERROR: YAML file does not exist'. This error indicates that CodeBuild couldn't find the buildspec.yml file in my GitHub repository, which makes sense because I hadn't created it yet.

I then opened my VSCode where I am connected to my EC2 instance, navigated to the root directory of my project, and created a buildspec.yml file with the below contents.

```
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto8
  pre_build:
    commands:
      - echo Initializing environment
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain nextwork --domain-owner 123456789012 --region ap-south-1 --query authorizationToken --output text` // replace 123456789012 with your actual account ID and ensure that the correct region is specified.

  build:
    commands:
      - echo Build started on `date`
      - mvn -s settings.xml compile
  post_build:
    commands:
      - echo Build completed on `date`
      - mvn -s settings.xml package
artifacts:
  files:
    - target/nextwork-web-project.war
  discard-paths: no
```

A screenshot of my buildspec.yml file is also shown below.

![Image](https://github.com/sumeet15n/CI-with-CodeBuild/blob/master/Screenshots/SS2.png)

I then ran the below commands in my VSCode terminal to push the changes to my GitHub repository.

```
git add .
git commit -m "Adding buildspec.yml file"
git push
```

I then navigated to my GitHub repository, refreshed the page, and confirmed that the buildspec.yml appeared in the root directory.

### Step 6: Verify successful build and artifacts

I navigated to my CodeBuild project 'nextwork-devops-cicd' and clicked on 'Retry build'. However, the status changed from in progress to failed again.

I navigated to the Phase details tab and noticed that the DOWNLOAD_SOURCE phase succeeded this time, but some other phases failed because the CodeBuild service role couldn't access the settings.xml file. To fix this, I granted CodeBuild's IAM role the permission to access CodeArtifact by attaching the policy named 'codeartifact-nextwork-consumer-policy' that I created in step 3.

I then navigated to my CodeBuild project 'nextwork-devops-cicd' and clicked on 'Retry build', the build was successful this time.

A screenshot of the success message is shown below.

![Image](https://github.com/sumeet15n/CI-with-CodeBuild/blob/master/Screenshots/SS3.png)

I also verified that my S3 bucket now contained the artifact zip file. The build process compiled the code from the source repository in GitHub, compressed it into a file, and stored it in the S3 bucket.

### Step 7: Delete resources

Finally, I deleted all the created resources after completing this project to avoid any further charges on AWS.

---

## Project Reflection

This project took me approximately 3 hours to complete. The most challenging part was resolving the build errors, and the most rewarding part was to see the final artifact file in my S3 bucket.

Big thanks to NextWork (https://www.nextwork.org/) for this project! I highly recommend this platform to anyone who wants to learn DevOps concepts and complete more projects like this one. Happy learning!