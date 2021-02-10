---
title: Apex PMD Code Analysis
layout: blog
category: [Salesforce, Best Practices]
excerpt: In this blog post, I will explain how to use PMD static code analysis CLI to analyze code assets in Salesforce.
comments: true
---

PMD Source Code Analysis CLI is most preferred choice when it comes to doing statis code analysis on Salesforce Apex Code Assets. In fact, it is recommended by Salesforce themselves.

## Pre-Requisites
1. Navigate to [PMD Website](https://pmd.github.io/){:target="\_blank"} and follow the installation procedure for your Operating System of choice.

## Steps to perform Code Analysis on your Salesforce Code Assets
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
12. Create a folder named "rulesets" for storing the rulesets
	``` bash
    mkdir rulesets
    ```
13. Create a folder named "AnalysisResults" for storing the code analysis results generated from the PMD CLI.
	``` bash
    mkdir AnalysisResults
    ```
14. Download all the PMD ruleset XML for Apex and organize it in a "rulesets" folder that created in Step 12.
	- [Best Practices](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/bestpractices.xml){:target="\_blank"}
	- [Code Style](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/codestyle.xml){:target="\_blank"}
	- [Design](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/design.xml){:target="\_blank"}
	- [Documentation](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/documentation.xml){:target="\_blank"}
	- [Code Smell](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/errorprone.xml){:target="\_blank"}
	- [Multithreading](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/multithreading.xml){:target="\_blank"}
	- [Performance](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/performance.xml){:target="\_blank"}
	- [Security](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/docs/security.xml){:target="\_blank"}
15. Open a command prompt on the root folder of your Project.
16. Execute following commands to generate the code analysis results in HTML Format.
	``` bash
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/bestpractices.xml -reportfile ./AnalysisResults/bestpractices.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/codestyle.xml -reportfile ./AnalysisResults/codestyle.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/design.xml -reportfile ./AnalysisResults/design.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/documentation.xml -reportfile ./AnalysisResults/documentation.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/errorprone.xml -reportfile ./AnalysisResults/errorprone.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/multithreading.xml -reportfile ./AnalysisResults/threading.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/performance.xml -reportfile ./AnalysisResults/performance.html
	pmd -d ./retrieved/unpackaged -f summaryhtml -R ./rulesets/security.xml -reportfile ./AnalysisResults/security.html
	```
17. You can also use the following commands to generate code analysis results in CSV Format. I usually prefer this.
	``` bash
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/bestpractices.xml -reportfile ./AnalysisResults/bestpractices.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/codestyle.xml -reportfile ./AnalysisResults/codestyle.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/design.xml -reportfile ./AnalysisResults/design.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/documentation.xml -reportfile ./AnalysisResults/documentation.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/errorprone.xml -reportfile ./AnalysisResults/errorprone.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/multithreading.xml -reportfile ./AnalysisResults/threading.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/performance.xml -reportfile ./AnalysisResults/performance.html
	pmd -d ./retrieved/unpackaged -f csv -R ./rulesets/security.xml -reportfile ./AnalysisResults/security.html
	```

Hurray ! We just analyzed all of our apex code assets in a matter of few minutes.
The next step is to go over each and every finding in the CSV and take corrective action on your code asset. Please do refer my blog on [Recommendations to fixing issues in Apex code](/recommendations-to-fixing-issues-in-apex-code/) to see how well we can fix the issues highlightd by PMD report.