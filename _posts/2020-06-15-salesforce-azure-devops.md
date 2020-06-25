---
title: Salesforce CI/CD using Azure DevOps Services
layout: blog
category: [Salesforce, Azure, DevOps]
excerpt: The biggest pain point for most developers and companies doing Salesforce development is to have a robust DevOps process. There are many costly tools and products for implementing DevOps in Salesforce. In this blog, I will elaborate on the DevOps process that you can setup using Azure DevOps services for free.
comments: true
---

## Getting the Fundamentals right
Before we dive in to the setup and configurations of the DevOps process, we should have a clear understanding of what Continuos Integration (CI) is and what Continuous Delivery (CD) is.

**Continuous Integration (CI)** is not a tool. It is a practice. 
- It is a practice for integrating the work of all developers into a common codebase.
- It involves running tests automatically when code is changed.
- It also involves detecting broken build & tests in few seconds to keep process flowing.

**Continuous Delivery (CD)** is not a tool. It is a practice.
- It involves propagating changes from DEV to PROD.
- It also involves propagating changes from PROD to DEV to keep orgs in sync.

## Understanding the current Development Model
As of today, Salesforce recommends two development models.
1. Package Development Model
2. Org Development Model

We need to set the deployment strategy in alignment with the development models.

In Package Development Model,
- Every artefact is bundled into a package and deployed separately
- Package should be self-sufficient & isolated
- Requires the organization to have adherence to architectural principles, existence of frameworks, horizontal & vertically sliced concerns addressed

Even though Package Development Model appears close to what we want to have, do not force yourself into package development if your organizational practices are not package based. Package Development Model would vision to release features as a package which is vertically and horizontally sliced. Any impacts that occur would only happen in that package/app and it would not affect the other packages/apps. Getting to the package development model maturity itself is a huge effort on its own.

In Org Development Model,
- Artefacts are bundled as functionalities/features for deployment (like a changeset bundle)
- Deployments are org based and not package based

Org Development Model is a common development model that most organizations follow. Every feature developed would be bundled into a changeset and be deployed to an org. Post deployment, QA would do a smoke testing and a regression testing just to ensure nothing in the org is impacted.

Now, let's define the problem statement.
> ABC Company uses Salesforce CRM for their business. They have a development team that develops functionalities as per business request in a Developer Sandbox. The team lead/team manager/deployment manager collects the list of items developed for each functionality from the developer, prepares changeset and deploys to QA Sandbox when it is ready for QA testing. After QA testing is pass, those functionalities are queued for PROD deployment which happens once every 2 weeks. The team lead/team manager/deployment manager again creates the changeset for PROD deployment and executes them every 2 weeks and initimates the organization stakeholders. For your information, they use **Azure** repositories for code management for their website development team.

### Key items to consider for deployment strategy
- It is clear that ABC company uses Org Development Model. So, we need to understand that anything related to package development model (even though it is fancy and attractive) should not be forced.
- They already use Azure repositories for version control needs. So, it is worth using Azure repo for SF code management.
- Azure has out of the box features for Devops which is called Azure DevOps services. This can be used as a tool for CI/CD.

### Things that we need before we start the DevOps configurations
1. Salesforce Developer Org [if you don't have one, signup and get a free personal dev org]
2. Developer Azure account [just go to dev.azure.com and register with your microsoft personal email address]
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
    - Connected App Name = "DevOps App"
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
In this step, we will create a project in Azure account.

1. Login to dev.azure.com
2. Create a new project named "SF DevOps" and save it.
    - Visibility should be Private
    - Version Control should be selected as Git
    - Work Item Process can be left with default option (Agile)
3. Open the project.
4. Click the "Repos" tab.
5. Click on "Generate Git Credentials". Copy the username and password somewhere safely. If you refresh this screen, you will not be able to see the password again. We will require this username & password when executing git commands. (I needed this step while configuring the DevOps in Ubuntu Linux machine. While trying out in Windows machine, it didn't even need this generated Git credentials.)

## Step 4. Getting the SF Org codebase to push to Azure Git repo
In this step, we will pull the codebase from salesforce and organize it in the way we want to version control it. Then, we will commit this code to the Azure Git repo we created in Step 3.

1. Create a folder named "Projects" in your local machine
2. Open the terminal/command prompt from this "Projects" folder.
3. Execute the following command for creating an empty SFDX project locally
    ``` bash
    sfdx force:project:create --projectname MyPersonalDevOrg
    ```
4. cd into the newly created project folder
    ``` bash
    cd MyPersonalDevOrg
    ```
5. Create a folder named "manifest"
    ```
    mkdir manifest
    ```
6. Type the following command to open the project in VS Code.
    ``` bash
    code .
    ```
7. Create a file named "package.xml" inside the manifest folder.
8. Open the "package.xml" file and paste the following contents. Essentially, we are saying that we are interested in Apex Classes, Apex Triggers, Visualforce Pages and Custom Objects only. You can add more metadata to "package,xml" file as per your needs.
    ``` xml
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
5. Create a folder named "buildfiles" using the following command
    ``` bash
    mkdir buildfiles
    ```
5. Paste the **server.key** inside the buildfiles folder
9. Execute the following command in terminal/command prompt. This will retrieve all the metadata from your SF org to the project folder in metadata format. Please do replace the xxx@xxx.com to your salesforce org username.
    ``` bash
    sfdx force:mdapi:retrieve -r retrieved -k manifest/package.xml -w 10 -u xxx@xxx.com
    ```
10. Unzip the retrieved metadata zip file into a folder named "retrieved". (**Note** that this unzip command may not work in Windows OS. So manually unzip.)
    ``` bash
    unzip retrieved/unpackaged.zip -d retrieved
    ```
11. Delete the zip file
    ``` bash
    rm retrieved/unpackaged.zip
    ```
12. Convert the unzipped "retrieved" folder contents from metadata format to SFDX format. This will add the metadata into the SFDX project structure under the folder "force-app"
    ``` bash
    sfdx force:mdapi:convert -r retrieved/unpackaged -d force-app
    ```
13. Execute the following command in terminal/command prompt to initialize the project folder for git tracking.
    ``` bash
    git init
    ```
14. Switch to VS Code. In .gitignore file, add the following lines to the last of the file. We are telling git that do not track "retrieved" and "toDeploy" folders as they are working directories.
    ```
    retrieved/
    toDeploy/
    ```
15. Add the Git user configurations. Execute the following commands in the terminal/command prompt. Replace "xxx@xxx.com" with your dev.azure.com login email address. Replace "testusername" with the username you had got in STEP 3 -> Point number 5.
    ``` bash
    git config user.email "xxx@xxx.com"
    git config user.name "testusername"
    ```
16. Add all the project files to git tracking by executing the following command in terminal/command prompt.
    ``` bash
    git add .
    ```
17. Execute the following command in terminal/command prompt to commit the code.
    ``` bash
    git commit -m "Initial Code"
    ```
18. Copy and execute the git remote command from the Azure Project's -> Repos page shown in the "Push an existing repository from command line" section. GIT_REPO_URL should be replaced with your azure git repo url.
    ``` bash
    git remote add origin GIT_REPO_URL
    ```
19. Execute the following command in terminal/command prompt to push the local project codebase to the Azure Git Repo. This might ask for the password. Provide the password that we had got from STEP 3 -> Point number 5.
    ``` bash
    git push -u origin master
    ```
20. After the command executes, switch to Azure Git Repo open in browser and refresh the page. You should see the Salesforce Org codebase in the repo.

## Step 5. Creating the Azure Pipeline for automated build & deploy
In this step, we will create the azure pipeline which will build, test and deploy the committed codebase from azure git repo to Salesforce Cloud. Here, we will use the latest SFDX CLI techniques to deploy instead of the old school ANT migration scripts.

1. In the Azure Project page open in the browser, navigate to "Pipelines" tab.
2. Click on "Create Pipeline"
3. Choose "Azure Repos Git" for "Where is your code?"
4. Choose "SF DevOps" project for "Select your repository"
5. Choose "Starter Pipeline" for "Configure your pipeline"
6. Overwrite the code shown in "Review your pipeline YAML" and paste the following code. (**Note**: a usual mistake that people do is to copy-paste the pipeline code and mess up the indentation of the yml code. If indentation is not right, you will have a tough time running the pipeline.)
    ```
    # Starter pipeline
    # Start with a minimal pipeline that you can customize to build and deploy your code.
    # Add steps that build, run tests, deploy, and more:
    # https://aka.ms/yaml

    trigger:
    batch: "true"
    branches:
        include:
        - master
    paths:
        exclude:
        - README.md
        - azure-pipelines.yml
    pr:
    autoCancel: "true"
    branches:
        include:
        - master
    paths:
        exclude:
        - README.md
    jobs:
    - job: Deploy
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master'))
    steps:
        - task: UseNode@1
        - bash: 
            npm install sfdx-cli --global
        displayName: Install Salesforce CLI
        - bash: 
            sfdx force:auth:jwt:grant --clientid $(salesforceDevOrgClientId) --jwtkeyfile ./buildfiles/server.key --username $(salesforceDevOrgUserName) --instanceurl $(salesforceDevOrgInstanceURL) -a devOrg
        displayName: Authorize salesforce org
        - bash: 
            sfdx force:source:convert -r ./force-app -d ./toDeploy
        displayName: Convert to deploy source
        - bash: 
            sfdx force:mdapi:deploy -l RunLocalTests -c -d ./toDeploy -u devOrg -w 10
        displayName: Run validation on source code
        - bash: 
            sfdx force:mdapi:deploy -l RunLocalTests -d ./toDeploy -u devOrg -w 10
        displayName: Deploy source code to Dev Org
    ```
    
    **Explanation of the pipeline code:**

    - We define 1 job called "Deploy" with a set of steps/tasks.
        1. Install SF CLI on the azure vm so that we can execute SFDX commands.
        2. Authorize the Dev Org and ensure that we can login using clientId, server.key and username
        3. Convert the SFDX code in "force-app" directory to metadata format for deployment
        4. Run a Validation using the converted code in metadata format using SFDX
        5. Deploy the converted code in metadata format using SFDX

7. Click on the "Variables" button and create the following variables
    - **salesforceDevOrgClientId** = paste the client id from the connected app we created in Step 2
    - **salesforceDevOrgInstanceURL** = https://login.salesforce.com
    - **salesforceDevOrgUserName** = type the username of your developer org
8. Click on "Save & Run". You will see the pipeline starting to run. You can click the build instance and see what the azure pipeline is doing while it is executing the commands. 
9. After the pipeline is executed, you can open the Salesforce Org in browser and navigate to Setup -> Environments -> Deploy -> Deployment Status. Here, you will see a recent Deployment Validation Success & a Deployment Success.
10. Run the following command in terminal/command prompt to pull the pipeline yml file to vscode locally.
    ``` bash
    git pull origin master
    ```

This completes the setup of Azure DevOps pipeline for automating Salesforce deployments.

**From now on, you just need to **
- open the project code in VS Code
- add/modify the code
- run the following commands to commit your code changes to azure git repo
    ``` bash
    git add .
    git commit -m "commit message"
    git push origin master
    ```
- the azure pipeline will automatically detect that a commit was made and it will automatically run the azure pipeline to deploy the code.
- even if the pipeline succeeds or fails, you will get an email notification regarding the latest build status.

## Moral of the Story
- You don't have to open Salesforce Org from now on for making code changes.
- Every code/metadata changes are made locally in VS Code and tracked by Azure Git version control. Version Control is the source of truth.
- Azure pipeline ensures that what is in GIT is there in the SF Org.
