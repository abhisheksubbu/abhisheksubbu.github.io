---
title: Install .NET Core SDK 3.1 in Ubuntu 20.04 LTS
layout: blog
category: [.NET]
excerpt: As of the release of Ubuntu 20.04 LTS, installation of dotnet core sdk has been a bit of pain. This blog shares the installation instructions clearly for dotnet-sdk-3.1 in Ubuntu 20.04 LTS
comments: true
---

This blog specifically describes the steps involved in installing .NET Core SDK 3.1 in Ubuntu 20.04 LTS Developer Machine.

1. Open a terminal and run the following commands to setup the 20.04 repositories

    ```bash
    sudo wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    ```

2. Install the .NET Core SDK
    ```bash
    sudo add-apt-repository universe
    sudo apt-get update
    sudo apt-get install apt-transport-https
    sudo apt-get update
    sudo apt-get install dotnet-sdk-3.1
    ```

> I would not recommend you to use the 19.04 repositories since the distribution has officially reached its [end of life](https://wiki.ubuntu.com/Releases){:target="\_blank"}.

#### Issues & Workarounds
- While executing the command `sudo apt-get install dotnet-sdk-3.1`, you may get dependency errors related to `libicu` not found in Ubuntu 20.04 LTS. If this is the case, then execute the following commands in terminal [courtesy: [Phillip Haydon](https://github.com/phillip-haydon){:target="\_blank"}]. Now try the `sudo apt-get install dotnet-sdk-3.1`, it should work.
    ```bash
    sudo wget http://mirrors.edge.kernel.org/ubuntu/pool/main/i/icu/libicu63_63.2-2_amd64.deb
    sudo dpkg -i libicu63_63.2-2_amd64.deb
    ```

![dotnet-core-list](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/dotnet-31-list.png)

You should see the [microsoft docs](https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-ubuntu-1804){:target="\_blank"} get updated in a day or two on linux package manager instructions specifically for Ubuntu 20.04 LTS.