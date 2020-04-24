---
title: SFDX Environment Preparation in Ubuntu 20.04 LTS
layout: blog
category: [.NET]
excerpt: In this blog, I will briefly explain the steps for setting up your SFDX environment in Ubuntu 20.04 LTS Linux machine
comments: true
---

### Pre-Requisites
1. Visual Studio Code - install it from the default Ubuntu Software Center

### Steps to configure SFDX
1. Install OpenJDK 11 using the following terminal command. Be patient since it takes some time.
    ```bash
    sudo apt install openjdk-11-jdk
    ```

2. Set the environment variables using the following terminal commands
    ```bash
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    echo $JAVA_HOME
    export PATH=$PATH:$JAVA_HOME/bin
    echo $PATH
    ```
3. Check the java version by typing the following command in terminal. It should show **openjdk version "11.0.7" 2020-04-14**
    ```bash
    java -version
    ```
4. In Visual Studio Code, open the Extensions tab and install the **Salesforce Extension Pack**
5. After the extension pack is installed, press the key combination Ctrl+Shift+P to open the command palette. Type **SFDX**. You should see the sfdx operations getting listed.

Great Job !! Your SFDX environment is setup in your local machine.