---
title: Salesforce CI/CD using Github
layout: blog
category: [Salesforce, Github, DevOps]
excerpt: The biggest pain point for most developers and companies doing Salesforce development is to have a robust DevOps process. There are many costly tools and products for implementing DevOps in Salesforce. In this blog, I will elaborate on the DevOps process that you can setup using Github for free.
comments: true
---

Please do read my blog on [Salesforce CI/CD with Azure DevOps Services](/salesforce-azure-devops/) to get upto speed on fundamental knowledge of CI, CD and devops practices to be employed for Salesforce. The same applies here as well. Just few steps change.

Now, let's define the problem statement.
> ABC Company uses Salesforce CRM for their business. They have a development team that develops functionalities as per business request in a Developer Sandbox. The team lead/team manager/deployment manager collects the list of items developed for each functionality from the developer, prepares changeset and deploys to QA Sandbox when it is ready for QA testing. After QA testing is pass, those functionalities are queued for PROD deployment which happens once every 2 weeks. The team lead/team manager/deployment manager again creates the changeset for PROD deployment and executes them every 2 weeks and initimates the organization stakeholders. For your information, they use **Github** repositories for code management for their website development team.

### Key items to consider for deployment strategy
- ABC company uses Org Development Model. So, we need to understand that anything related to package development model (even though it is fancy and attractive) should not be forced.
- They already use Github repositories for version control needs.
- Github has out of the box features for Devops which is called Github Actions. This can be used as a tool for CI/CD.

### Things that we need before we start the DevOps configurations
1. Salesforce Developer Org [if you don't have one, signup and get a free personal dev org]
2. Personal Github Account
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

**Solution:** The issue is with the path of the jwtkeyfile that is specified in the command. Ensure that the server.key file exists in the path that you have specified in the command. And more importantly, checkin the code so that Github pipeline can find the file on the Github repo.


## Step 3: Repository Setup in Github
In this step, we will create a new repo in Github.

1. Login to Github
2. Create a new repository named "SFDevOps" and save it.
    - Visibility should be Public/Private
3. Click on **Create Repository** button

## Step 4. Getting the SF Org codebase to push to Github repo
In this step, we will pull the codebase from salesforce and organize it in the way we want to version control it. Then, we will commit this code to the Github's newly created repo we created in Step 3.

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
15. Add the Git user configurations. Execute the following commands in the terminal/command prompt. Replace "xxx@xxx.com" with your github login email address. Replace "testusername" with the github username.
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
18. Copy and execute the git remote command from the Github's new repo page. GIT_REPO_URL should be replaced with your git repo url.
    ``` bash
    git remote add origin GIT_REPO_URL
    ```
19. Execute the following command in terminal/command prompt to push the local project codebase to the Github Repo. This might ask for the password. Provide the password of your Github login.
    ``` bash
    git push -u origin master
    ```
20. After the command executes, switch to Github Repo open in browser and refresh the page. You should see the Salesforce Org codebase in the repo.

## Step 5. Creating the Github Workflow for automated build & deploy
In this step, we will create the github pipeline which will build, test and deploy the committed codebase from github repo to Salesforce Cloud. Here, we will use the latest SFDX CLI techniques to deploy instead of the old school ANT migration scripts.

1. In the Github repo page open in the browser, navigate to "Actions" tab.
2. In the "Choose the starter worklfow" page, click on the "Setup this workflow" button in the Simple Workflow box.
4. In the editor, rename the pipeline name from blank.yml to pipeline.yml.
6. Clear the code and paste the following code from my github repo. (**Note**: a usual mistake that people do is to copy-paste the pipeline code and mess up the indentation of the yml code. If indentation is not right, you will have a tough time running the pipeline.)
    [Refer the actual pipeline.yml code here](https://github.com/abhisheksubbu/DevMainAzureGithub/blob/master/.github/workflows/pipelines.yml)
    
    **Explanation of the pipeline code:**

    - We define 1 job called "build" with a set of steps/tasks.
        1. Instruct the pipeline to provision a VM with "ubuntu-latest" os.
        1. Install SF CLI on the vm so that we can execute SFDX commands.
        2. Authorize the Dev Org and ensure that we can login using clientId, server.key and username. Note that here we pick the variables values from environment settings in Github.
        3. Convert the SFDX code in "force-app" directory to metadata format for deployment
        4. Run a Validation using the converted code in metadata format using SFDX
        5. Deploy the converted code in metadata format using SFDX
7. Commit the yml file using a commit on Github.
7. Navigate to the Github repo page and go to "Settings" tab.
7. Click "Secrets" nav item on the left menu and then click on the "New Secret" button and create the following environment variables
    - **SALESFORCEPRODCLIENTID** = paste the client id from the connected app we created in Step 2
    - **SALESFORCEPRODINSTANCEURL** = https://login.salesforce.com
    - **SALESFORCEPRODUSERNAME** = type the username of your developer org
10. Run the following command in terminal/command prompt to pull the pipeline yml file to vscode locally.
    ``` bash
    git pull origin master
    ```

This completes the setup of Github DevOps pipeline for automating Salesforce deployments.

**From now on, you just need to**
- open the project code in VS Code
- add/modify the code
- run the following commands to commit your code changes to github repo
    ``` bash
    git add .
    git commit -m "commit message"
    git push origin master
    ```
- the github pipeline will automatically detect that a commit was made and it will automatically run the pipeline to deploy the code.
- even if the pipeline succeeds or fails, you will get an email notification regarding the latest build status.

## Moral of the Story
- You don't have to open Salesforce Org from now on for making code changes.
- Every code/metadata changes are made locally in VS Code and tracked by Github version control. Version Control is the source of truth.
- Github pipeline ensures that what is in GIT repo is there in the SF Org.
