---
title: Salesforce CI/CD using Azure DevOps Services and Github Source Control
layout: blog
category: [Salesforce, Azure, DevOps, Github]
excerpt: In this blog, I detail out the nuts and bolts needed to do implement Salesforce DevOps with Github Version Control and Azure Pipelines.
comments: true
---

### Things that we need before we start the DevOps configurations
1. Salesforce Developer Orgs (to simulate dev, qa, uat and prod) [if you don't have one, signup and get a free personal dev org]
2. Developer Azure account [just go to dev.azure.com and register with your microsoft personal email address]
3. Github Account [just go to github.com and register with your personal email address]
3. Salesforce CLI installed locally
4. Visual Studio Code installed locally with "Salesforce Extension Pack" installed.
5. Git (either via npm or as installer)

## Step 1: Certificates and Key
The first step is to create a self signed certificate and private key that we need for configuring the DevOps process to authorize with Salesforce org.

> If your operating system is Windows, you have to install OpenSSL before you attempt these command. In linux, you don't have to install anything. Just execute the commands.

1. In your terminal/command prompt, type the following command. This creates the private key named '**server.key**'.
```
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key
```
2. Next, type the following command in terminal/command prompt to generate the '**server.cs**r' file.
```
openssl req -new -key server.key -out server.csr
```
3. Now, type the following command in terminal/command prompt for generate the '**server.crt**' certificate.
```
openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
```

## Step 2: Connected App Setup in Salesforce for Devops Process
In this step, we will create a new connected app for DevOps process to authorize using the certificate that we created earlier.
1. Login in to Salesforce Developer Org
2. Navigate to Setup -> Apps -> App Manager
3. Create a new Connected App with the following details and save it.
    - Connected App Name = "Salesforce DevOps App"
    - Contact Email = specify your personal email address
    - Enable OAuth Settings = tick mark it to checked state
    - Callback URL = http://localhost:1717/OauthRedirect
    - Use digital signatures = tick mark it to checked state
        - Browse and select the **server.crt** file from your local machine
    - Selected OAuth scopes
        - Access and manage your data (api)
        - Access your basic information (id, profile, email, address, phone)
        - Perform requests on your behalf at any time (refresh_token, offline_access)
        - Provide access to your data via the Web (web)
    - Require Secret for Web Server Flow = tick mark it to checked state
4. Click the "Manage" button the connected app, set the following and save.
    - Permitted Users = Admin approved users are pre-authorized
5. After saving the permitted users, scroll down to "Profiles" related list and click the "Manage Profiles" button. Add the "System Administrator" profile or equivalent profile that your DevOps user is setup with.

**Test if SFDX authorization to SF org is successfull or not.** On executing the force:auth:jwt:grant command, it should say "Successfully authorized xxx@xxx.com with org ID 00D2x000000aBcDEAM". If this fails, then either the ClientId copied & pasted is not proper or certificate/key file is not generated properly from OpenSSL. Ensure to replace the username, client id with you own values.

``` bash
sfdx force:auth:jwt:grant --clientid 3MVG97quAmFZJfVyzexU2c1VnTmNIkZ5g1IwJ_abcd_menLDWTuYasRhsInZHkA.Jfw.BmI4rbHYmjdzZBeqC --jwtkeyfile server.key --username xxx@xxx.com --instanceurl https://login.salesforce.com
```

#### Possible Errors
> ERROR running force:auth:jwt:grant:  We encountered a JSON web token error, which is likely not an issue with Salesforce CLI. Here's the error: invalid assertion

**Solution:** Generate the server.key, csr and crt again using OpenSSL. Update the crt in the Connected App in Salesforce. Then update the key file in the buildfiles folder in the vscode project. Checkin the code and this problem should be resolved.

> ERROR running force:source:deploy: We encountered a JSON web token error, which is likely not an issue with Salesforce CLI. Here's the error: ENOENT: no such file or directory, open '<pathtojob>\DevSFDX\ELDEV@tmp\secretFiles\f5e59c25-44df-4273-ae1f-05d927940543\server.key'

**Solution:** The issue is with the path of the jwtkeyfile that is specified in the command. Ensure that the server.key file exists in the path that you have specified in the command. And more importantly, checkin the code so that Azure pipeline can find the file on the Azure git repo.


## Step 3: Project Setup in Azure
In this step, we will create a project in Azure account just to configure the YAML pipelines. We will not use the Repos in Azure.
1. Login to dev.azure.com
2. Create a new project named "Salesforce" and save it.
    - Visibility should be Private
    - Version Control should be selected as Git
    - Work Item Process can be left with default option (Agile)
3. Open the Azure Project and navigate to the "Library"
    - Here, go to "Secure Files" and upload the "server.key" file.
    - In the Variable Groups, create separate variable groups for dev, qa, uat and prod (all environments that apply for your org). - Ensure that you name the variable group in the following format sfdx-org-{env} (example: sfdx-org-dev, sfdx-org-qa, etc)
        - The variable group needs to have 4 variables namely Clientid, InstanceUrl, OrgAlias, SalesforceUserName and update corresponding values in these variables. We will reference these environment specific variable groups from the pipeline based on the branch a PR or commit is made.

| Variable Group | Variables & its Values|
|----------------|-----------------------|
| sfdx-org-dev   | Clientid = {paste the client id from the connected app we created in Step 2}, InstanceUrl=https://test.salesforce.com/, OrgAlias=dev, SalesforceUserName={username of the SF user}|
| sfdx-org-qa   | Clientid = {paste the client id from the connected app we created in Step 2}, InstanceUrl=https://test.salesforce.com/, OrgAlias=qa, SalesforceUserName={username of the SF user}|
| sfdx-org-uat   | Clientid = {paste the client id from the connected app we created in Step 2}, InstanceUrl=https://test.salesforce.com/, OrgAlias=uat, SalesforceUserName={username of the SF user}|
| sfdx-org-prod   | Clientid = {paste the client id from the connected app we created in Step 2}, InstanceUrl=https://login.salesforce.com/, OrgAlias=prod, SalesforceUserName={username of the SF user}|

## Step 4: Project Setup in Github
In this step, we will create a project in Github for managing repositories and branches.
1. Login to github.com
2. Create a Project in github named "Salesforce". This will create "Salesforce" as a Repository with 1 branch named "master".

## Step 5. Create an SFDX Project and push to Github repo
In this step, we will push a blank SFDX project in VS Code and commit to the master branch in Github.

1. First, clone the Github project locally.
    ```
    git clone <URL of the Repo in Github>
    ```
2. Open the terminal/command prompt from this "Salesforce" folder.
3. Execute the following command for creating an empty SFDX project locally
    ``` bash
    sfdx force:project:create --projectname Salesforce
    ```
4. cd into the newly created project folder
    ``` bash
    cd Salesforce
    ```
5. Add the Git user configurations. Execute the following commands in the terminal/command prompt. Replace "xxx@xxx.com" with your dev.azure.com login email address. Replace "testusername" with the username with which you login to Github.
    ``` bash
    git config user.email "xxx@xxx.com"
    git config user.name "testusername"
    ```
6. Add all the project files to git tracking by executing the following command in terminal/command prompt.
    ``` bash
    git add .
    ```
7. Execute the following command in terminal/command prompt to commit the code.
    ``` bash
    git commit -m "Initial Code"
    ```
8. Copy and execute the git remote command. GIT_REPO_URL should be replaced with your GITHUB repo url.
    ``` bash
    git remote add origin GIT_REPO_URL
    ```
9. Execute the following command in terminal/command prompt to push the local project codebase to the Github Repo. This might ask for the password. This password most probably will be your Github Personal Access Token and not the normal github password.
    ``` bash
    git push -u origin master
    ```
10. After the command executes, go to Github Repo in the browser and refresh the page. You should see the blank sfdx project codebase in the "Salesforce" repo on master branch.

## Step 6. Creating branches in Github for lower environments
1. On the browser, open the "Salesforce" repo in Github
2. Ensure that the branch shows "master"
3. Click on the "master" branch dropdown and type" dev" in the "Find or create a branch" textbox and hit Enter. This will create a dev branch from the master branch
4. Again switch the selection in the branch dropdown to "master" and repeat the Step 3 to create "qa" and "uat" branch from master.

## Step 7. Creating a feature branch to pump YAML pipeline capability upstream
1. In your local machine where you had cloned the "Salesforce" Github repo, open the terminal/CMD from the "Salesforce" project folder and type the following command. This will ensure that your local "Salesforce" codebase is up to date with Github repo.
    ```
    git pull
    ```
2. Switch to "dev" branch. This will switch your working branch from master to dev
    ```
    git switch dev
    ```
3. Once switched to "dev" branch, create a new feature branch for YAML changes to be committed. Executing this command will create a feature branch named "yaml-changes" locally from the "dev" branch code.
    ```
    git checkout -b yaml-changes
    ```
4. Right click on the "force-app" folder and select the option "SFDX: Generate Manifest File". Specify the name of the file as "package.xml". It will create a "package.xml" file inside a new folder named "manifest".
5. Open the “package.xml” file and paste the following contents. Essentially, we are saying that we are interested in Apex Classes, Apex Triggers, Visualforce Pages and Custom Objects only. You can add more metadata to “package,xml” file as per your needs.
    ```xml
    <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <Package xmlns="http://soap.sforce.com/2006/04/metadata">
        <types>
            <members>*</members>
            <name>ApexClass</name>
        </types>
        <types>
            <members>*</members>
            <name>ApexTrigger</name>
        </types>
        <types>
            <members>*</members>
            <name>ApexPage</name>
        </types>
        <types>
            <members>*</members>
            <name>CustomObject</name>
        </types>
        <version>44.0</version>
    </Package>
    ```
6. Use VS Code Command Palette and do an "SFDX Authorize an Org". Make sure you are authorizing against the right dev or pre-dev sandbox to pull the metadata from.
7. Once package.xml is saved, right click on the file and choose "SFDX: Retrieve Source in Manifest from Org"
8. Create a folder named "pr"
    ```
    mkdir pr
    ```
9. From VS Code, add a file named "pr-build-dev.yml" inside the pr folder and paste the following YAML code. This pipeline will be used to do a validation check on DEV Sandbox when anyone raises a PR from feature branch to dev branch.
    ```
    # Salesforce Build Pipeline

    trigger: none

    pr:
    - dev

    jobs:
    - job: Validate
    variables:
    - group: sfdx-org-dev

    steps:
        - task: UseNode@1
        displayName: 'Use Node.js 16.15.0'
        inputs:
            version: '16.15.0'
        - bash: 
            npm install sfdx-cli --global
        displayName: Install Salesforce CLI
        - task: DownloadSecureFile@1
        name: jwtKey
        displayName: 'Download Server JWT Key'
        inputs:
            secureFile: 'server.key'
        - bash: 
            sfdx force:auth:jwt:grant --clientid $(Clientid) --jwtkeyfile $(jwtKey.secureFilePath) --username $(SalesforceUserName) --instanceurl $(InstanceUrl) -a $(OrgAlias)
        displayName: Authorize salesforce org
        - bash: 
            sfdx force:source:deploy -l RunLocalTests -c -x manifest/package.xml -u $(OrgAlias) -w 10
        displayName: Run Validation on source code
    ```
10. Similary, you can crate similar YAML files for other environments (qa, uat and prod). Let's say you want validation check to happen when someone raises a PR to QA Branch. So, in this case, create a file named "pr-build-qa.yml" and use the previous YAML code and change all "dev" words to "qa".
11. On the project folder level, add a new file named "build.yml". This pipeline will take the responsibility of deploying the committed code to respective salesforce sandbox or production org dynamically. Add the following yml code in this file.
    ```
    # Salesforce DevOps Pipeline

    trigger:
    batch: "true"
    branches:
        include:
        - master
        - dev
        - qa
        - uat
    paths:
        exclude:
        - README.md
        - azure-pipelines.yml
        - pr/pr-build-dev.yml
        - pr/pr-build-qa.yml
        - pr/pr-build-uat.yml
        - pr/pr-build-master.yml
    pr: none

    jobs:
    - job: Deploy
    variables:
    - ${{ if eq(variables['build.SourceBranchName'], 'dev') }}:
        - group: sfdx-org-dev
    - ${{ if eq(variables['build.SourceBranchName'], 'qa') }}:
        - group: sfdx-org-qa
    - ${{ if eq(variables['build.SourceBranchName'], 'uat') }}:
        - group: sfdx-org-uat
    - ${{ if eq(variables['build.SourceBranchName'], 'master') }}:
        - group: sfdx-org-master
    steps:
        - task: UseNode@1
        displayName: 'Use Node.js 16.15.0'
        inputs:
            version: '16.15.0'
        - bash: 
            npm install sfdx-cli --global
        displayName: Install Salesforce CLI
        - task: DownloadSecureFile@1
        name: jwtKey
        displayName: 'Download Server JWT Key'
        inputs:
            secureFile: 'server.key'
        - bash: 
            sfdx force:auth:jwt:grant --clientid $(Clientid) --jwtkeyfile $(jwtKey.secureFilePath) --username $(SalesforceUserName) --instanceurl $(InstanceUrl) -a $(OrgAlias)
        displayName: Authorize salesforce org
        - bash: 
            sfdx force:source:deploy -l RunLocalTests -c -x manifest/package.xml -u $(OrgAlias) -w 10
        displayName: Run Validation on source code
        - bash: 
            sfdx force:source:deploy -l RunLocalTests -x manifest/package.xml -u $(OrgAlias) -w 10
        displayName: Deploy source code to Salesforce Org
    ```
12. Add all the project files to git tracking by executing the following command in terminal/command prompt.
    ``` bash
    git add .
    ```
13. Execute the following command in terminal/command prompt to commit the code.
    ``` bash
    git commit -m "YAML and Metadata Added"
    ```
14. Execute the following command in terminal/command prompt to push the local project codebase to the github Repo.
    ``` bash
    git push -u origin yaml-changes
    ```
15. After the command executes, go to Github Repo in the browser and refresh the page. You should see the committed codebase in the repo on yaml-changes branch.

## Step 8. Creating the Azure Pipeline for automated build
In this step, we will create the azure pipeline which will use the PR YAML in Github to do validation to Salesforce.
1. Navigate to Azure DevOps and open the "Salesforce" project
2. Navigate to "Pipelines"
3. Click on "New Pipeline"
    - Choose "Github" as our code is in Github
    - Choose the "Salesforce" repository
    - Choose "Existing Azure Pipelines YAML File"
    - Choose Branch as "yaml-changes"
    - Choose Path as "/pr/pr-build-dev.yml"
    - Click Continue. This will create the Pipeline in Azure that will fire when a PR is initiated from any Feature Branch to Dev Branch.
4. Similarly, repeat Step 3 for adding PR pipeline for QA, UAT and PROD
5. Ensure that the pipeline named are renamed to something that makes sense to you. (For example: PR Validate to DEV, etc)

## Step 9. Creating the Azure Pipeline for automated deploy
In this step, we will create the azure pipeline which will use the YAML in Github to do deploy to Salesforce.
1. Navigate to Azure DevOps and open the "Salesforce" project
2. Navigate to "Pipelines"
3. Click on "New Pipeline"
    - Choose "Github" as our code is in Github
    - Choose the "Salesforce" repository
    - Choose "Existing Azure Pipelines YAML File"
    - Choose Branch as "yaml-changes"
    - Choose Path as "/build.yml"
    - Click Continue. This will create the Pipeline in Azure that will fire when a PR is approved/direct commit is detected to DEV, QA, UAT and MASTER Branch.
4. Ensure that the pipeline named are renamed to something that makes sense to you. (For example: Deploy to Salesforce Org)

This completes the setup of Azure DevOps pipeline for automating Salesforce build and deployments with Github Source Control.


# Best Practice
- Always create a feature branch for your new/modified changes from dev
- Raise PR to DEV
- Let the PR Validate Pipeline Pass
- If the Validate checks pass, let the Team Lead do a code review + approve the PR
- Approving the PR will fire the Deploy Pipeline to deploy the code to DEV sandbox
- Promote your changes from DEV to QA using a PR
    - This PR will cause the "PR Validate to QA" pipeline to fire
    - Now, TL reviews and approves the PR
    - Deploy Pipeline deploys code to QA Sandbox
- Similarly, promote your changes to UAT and PROD using PR
- Never do a direct commit on DEV, QA, UAT or PROD. Always promote upstream from the feature branch.

## Moral of the Story
- Since Github is acquired by Microsoft, the integration between these 2 systems are smooth.
- Developers are more comfortable with Github (into which you manage the repo, branch and commits)
- Azure takes care of all the build and deploy process automatically
- This setup is dynamic enough to do the validation & deploy on the right salesforce environment
- Pipelines works based on the package.xml. Hence you have control of what is validated and deployed against an org.
