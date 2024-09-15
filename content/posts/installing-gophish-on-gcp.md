---
title: "Installing Gophish on Google Cloud Platform: A Step-by-Step Guide"
date: 2024-07-19T00:00:00+00:00
# weight: 1
# aliases: ["/first"]
categories: ["cybersecurity", "gophish", "Google Cloud Platform"]
tags: ["cybersecurity", "gophish", "Google Cloud Platform"]
author: "Tomi Mulhartono"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

## What is Gophish?

Gophish is a powerful open-source phishing framework designed to make the process of conducting and managing phishing campaigns as straightforward as possible. Think of it as your go-to tool for simulating real-world phishing attacks. The best part? It's free and easy to use!

>- **Affordable:** Gophish is open-source software, which means you don't have to pay a cent to use it.
>- **Accessible:** Written in the Go programming language, Gophish releases are compiled binaries with no dependencies. This means installation is as easy as "download and run."

---

## Deploying Gophish on GCP

Alright, let's dive into how you can get Gophish up and running on Google Cloud Platform (GCP).

### Step 1: Create a Google Compute Engine Instance

First things first, we need a virtual machine (VM) on Google Cloud Platform. Go to your GCP console and create a new Google Compute Engine instance. Choose the "**N1**" machine type and stick with the default settings.

![Gophish](/images/gophish-1.jpg)

![Gophish](/images/gophish-2.jpg)


### Step 2: SSH into Your VM

Once your VM is set up, you'll need to connect to it. Use the SSH feature in the GCP console to access your VM, and then run these commands:

{{< highlight bash >}}

sudo adduser -c "Gophish" gophish
sudo su - gophish
curl -L https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-linux-64bit.zip -o gophish-v0.11.0-linux-64bit.zip
mkdir gophish-v0.11.0
cd gophish-v0.11.0
sudo apt-get install unzip
unzip ../gophish-v0.11.0-linux-64bit.zip
rm ../gophish-v0.11.0-linux-64bit.zip

{{< /highlight >}}

Here's a quick breakdown of what these commands do:

1. We create a new user called "gophish" and switch to that user.
2. We download the Gophish zip file from GitHub using ``curl``.
3. We create a directory for Gophish and unzip the downloaded file into it.


### Step 3: Exploring the Extracted Files

Once the zip file is extracted, you'll find several files, but the most important one is ``config.json``.

![Gophish](/images/gophish-3.jpg)

**Configuring config.json**

The ``config.json`` file contains settings for two servers:

- **admin_server:** This server hosts the admin console where you can create and manage phishing campaigns. By default, it runs on "**127.0.0.1:3333**", but we'll change this to "**0.0.0.0:3333**" so it can be accessed from anywhere.
- **phish_server:** This server hosts the phishing landing pages. By default, it runs on "**0.0.0.0:80**", but we'll change this to "**0.0.0.0:5555**" to avoid issues with privileged ports.

![Gophish](/images/gophish-4.jpg)

### Step 4: Adding Firewall Rules in GCP

Next, we need to create firewall rules to allow traffic to our Gophish instance on ports 3333 and 5555.

**Step-by-Step Firewall Configuration**

> 1. Go to **[Google Cloud Console.](https://console.cloud.google.com/)**
> 2. From the left navigation menu, select **VPC Network** and then **Firewall rules.**
> 3. Click the **Create Firewall Rule** button.
> 4. Configure the Firewall Rule.
> 5. Click **Create** to save the firewall rule.

**Example Configuration:**

| Parameter | Details |
| --- | --- |
| Name | Give the firewall rule a name, like "**allow-gophish-ports**". |
| Network | Select the appropriate network, usually "**default**". |
| Priority | Leave default or set a priority as needed. |
| Direction of Traffic | Select **Ingress**. |
| Action on Match | Select **Allow**. |
| Targets | Choose **All instances in the network** or restrict to specific VMs using tags. |
| Source IP Ranges | Enter "**0.0.0.0/0**" to allow access from all IP addresses, or restrict to specific IP ranges if needed. |
| Protocols and Ports | Select **Specified protocols and ports** and enter "**tcp:3333,5555**"". |

![Gophish](/images/gophish-5.jpg)

---

## Starting the Gophish Service
Now that everything is set up, we can start the Gophish service. Run the following command in your SSH session:

{{< highlight bash >}}

sudo ./gophish

{{< /highlight >}}

![Gophish](/images/gophish-6.jpg)

### Accessing Gophish
To access the Gophish admin console, open your web browser and navigate to "**https://<YOUR_VM_IP>:3333**". You'll see the Gophish login screen:

![Gophish](/images/gophish-7.jpg)

### Logging In
When you start Gophish, it will display the default login credentials in the terminal. Use these credentials to log in.

> **Note:** Gophish takes security seriously. The default password is unique for each deployment, and you'll be prompted to change it immediately after your first login.

### The Gophish Admin Console
Once you're logged in, you'll be greeted by the Gophish dashboard. Here's a quick rundown of the main sections:

- **Dashboard:** Provides an overview of your campaigns and their performance.
- **Campaigns:** Create and manage phishing campaigns tailored to different teams or purposes.
- **Users and Groups:** Add user emails and group them. You can also import emails from a CSV file.
- **Email Templates:** Design and manage phishing email templates.
- **Landing Pages:** Set up landing pages and track user interactions.
- **Sending Profiles:** Configure sender details for your phishing emails.

Here's a sneak peek at the Gophish admin console:

![Gophish](/images/gophish-8.jpg)

---

## Conclusion
And that's it! You've successfully set up Gophish on Google Cloud Platform. With Gophish, you can easily create and manage phishing campaigns to train your team and improve your organization's security awareness.

Gophish is a fantastic option for small businesses with tight budgets, offering powerful features without the hefty price tag of other solutions. Plus, it's super easy to use!

