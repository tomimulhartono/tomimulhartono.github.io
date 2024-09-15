---
title: "Getting Started with Your New Application on OpenResty Edge: An Easy-to-Follow Tutorial"
date: 2024-09-06T00:00:00+00:00
# weight: 1
# aliases: ["/first"]
categories: ["cybersecurity", "OpenResty", "Web Application Firewall"]
tags: ["cybersecurity", "OpenResty", "Web Application Firewall"]
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

Welcome aboard! If you’re gearing up to launch a new application on OpenResty Edge, you’re in the right place. Whether you’re a seasoned pro or just starting out, I’ll walk you through every step of the way with a dash of fun. Grab your favorite drink, settle in, and let’s dive into this setup adventure!

---

## Creating Your New Application

Let's get that new app live and kicking in no time. Here's how to create your first application on OpenResty Edge.

1. **Login** to your OpenResty Edge dashboard

Head over to the OpenResty Edge dashboard, and log in using your credentials.

![Screenshoot](/images/openresty-1.jpg)

2. Navigate to the **Applications** section 

Once you're logged in, take a look at the top menu bar and click on **Applications**. This is where the magic begins! Once you're in the applications section, click on the **Create New App** button to start the process of building your app.

![Screenshoot](/images/openresty-2.jpg)

3. Fill Out the application details

You'll need to provide some information to configure your app. Here’s a quick breakdown of each field you’ll encounter:

| Fields | What to Enter |
| --- | --- | 
| **Domains** | Enter your domain name here (e.g., `tomi-example.co.id`). |
| **Partitions** | 	Select **Default**. Keep it simple for now. |
| **Default Application** | Leave this blank unless you have something specific in mind. |
| **HTTP Port** | You can leave this blank for now. |
| **HTTPS Port** | 	Set to `443` for secure traffic. |
| **HTTP2 Status (HTTPS)** | No need to select unless you want HTTP2 support. |
| **Label** | Give your app a memorable label, like `Create a New Example App`. |
| **Proxy to Upstream** | Leave this off for now; you can enable it later. |
| **Log Format** | You can skip this for now. I’ll keep it simple. |
| **Log File** | No need to fill this in unless you have specific logging requirements. |
| **Load SSL Certificate by IP Addresses** | Don’t check this unless you’re specifically using IP-based SSL certificates. |
| **Load Application by IP Addresses** | Uncheck this unless your setup requires IP-based application loading. |

![Screenshoot](/images/openresty-3.jpg)

Once you’ve filled everything out, hit **Save**! Your application is officially created, and you're one step closer to deploying it live.

---

## Configuring Page Rules

Next up, let's make sure everything runs smoothly by configuring **Page Rules**. This allows you to set conditions and actions for traffic coming to your application.

### Custom Edge Language Rules at the Beginning of This Page

Before diving into predefined rules, let’s look at how you can add custom rules using OpenResty's Edge Language. For example, if you want to block requests from specific IPs and log alerts, you can use the following:

{{< highlight bash >}}

client-addr !~~ any (192.168.100.1) =>
    errlog(level: "alert", "SEC-00- Unauthorized IP!", foreign-call(module: "headers-log", func: "go") ~ req-body()),
    exit(444);

{{< /highlight >}}

![Screenshoot](/images/openresty-4.jpg)

This rule does two things:
- Blocks traffic from the IP `192.168.100.1`.
- Logs an alert with the message "**Unauthorized IP!**".

Feel free to tweak these rules to fit your specific needs. After adding them, hit **Save** to apply.

### Create New Rules

Let’s move on to setting up some common rules you'll probably need, starting with redirecting traffic from HTTP to HTTPS.

![Screenshoot](/images/openresty-10.jpg)

#### Redirect HTTP to HTTPS

Redirecting users from HTTP to HTTPS is a great way to improve both security and user experience. Here’s how you can set that up:

1. Click on **New Rule** in the Page Rules section.

2. Set the condition to identify when the rule should trigger:

- In the **When** section, click **Add a new condition**.
- From the **Variable** dropdown, choose **Scheme**.
- In the **Operator** dropdown, select **String =**.
- In the **Value** dropdown, select **http**.

![Screenshoot](/images/openresty-5.jpg)

3. Add the Action to redirect traffic:

- Under the **Actions** section, click **Add a new action**.
- From the Action Types dropdown, **choose Redirect**.
- Set the following fields:
    - **URI:** Choose **current**.
    - **URI arguments**: Choose **current**.
    - **Scheme:** Set this to **https**.
    - **Host:** Choose **current**.
    - **Port:** Set this to **blank value**.
    - **Status Code:** Choose **302 Moved Temporarily**.

![Screenshoot](/images/openresty-6.jpg)

This will ensure all users trying to access your site via HTTP will be securely redirected to HTTPS.

#### Set Proxy Header

If your application relies on specific headers to proxy traffic correctly, you’ll want to configure that as well. Let’s set up a rule to ensure the correct host header is passed along:

1. Click on **New Rule** in the Page Rules section.

2. Set the condition for when this rule should apply:

- In the **When** section, click **Add a new condition**.
- From the **Variable** dropdown, choose **Host**.
- In the **Value** field, choose **String**, then type your domain, e.g., `tomi-example.co.id`.

![Screenshoot](/images/openresty-7.jpg)

3. Set the first action to modify the **Host** header:

- Under the **Actions** section, click **Add a new action**.
- From the **Action Types** dropdown, choose **Set proxy header**.
- Set the following fields:
    - **Header:** Enter **Host**.
    - **Value:** Choose **String** and enter your domain, e.g., `tomi-example.co.id`.

4. Add a second action to append the **X-Forwarded-For** header:

- Under the **Actions** section, click **Add a new action** again.
- From the **Action Types** dropdown, choose **Append proxy header value**.
- Set the following fields:
    - **Header:** Choose **X-Forwarded-For**.
    - **Value:** Choose **Built-in Variable** and select **Client address**.

![Screenshoot](/images/openresty-8.jpg)

5. Configure the **Proxy** settings:

- Scroll down to the **Proxy** section to **On**.
- Select the appropriate **Proxy to upstream** from the dropdown, e.g., `example-backend`.
- For **Balancing Policy**, choose **Round Robin**

![Screenshoot](/images/openresty-9.jpg)

This ensures the correct headers are sent to the backend and helps maintain a smooth user experience.

### Add Custom Edge Language Rules at the End of This Page

To improve site performance, caching static assets like JavaScript and CSS is a great practice. Here’s how you can set up cache rules:

{{< highlight bash >}}

uri-suffix(
rx:s/\.(?:js|css|xml).{0,3}$/),
req-method("GET")=>
    enable-proxy-cache(key: uri),
    enforce-proxy-cache(7 [day]),
    set-resp-header("Cache-Control", "public, max-age=604800, immutable");
uri-suffix(
rx:s/\.(?:gif|png|js|css|html|jpg|wof|svg|woff2|ttf|otf|eot|ico).{0,5}$/),
req-method("GET")=>
    enable-proxy-cache(key: uri),
    enforce-proxy-cache(365 [day]),
    set-resp-header("Cache-Control", "public, max-age=3153600, immutable");

{{< /highlight >}}

![Screenshoot](/images/openresty-11.jpg)

This speeds up load times by storing frequently requested assets for a longer period.

---

## Configuring Upstreams

Now, let’s make sure your app knows where to send requests by configuring an upstream server (where your app's backend lives).  

1. Click on **Upstreams** from your dashboard menu.

2. Add a **New Upstream** and fill out the details:

| Fields | What to Enter |
| --- | --- | 
| **Upstream Name** | Enter a name for your upstream, like `example-backend`. |
| **Protocol** | Choose `HTTPS`. |
| **Host** | Enter the domain or IP address of your backend server (e.g., `tomi-backend.int`). |
| **Port** | Set this to `443`. |
| **Weight** | Set it to `1`. |
| **Enabled** | Switch this `On`. |

![Screenshoot](/images/openresty-12.jpg)

![Screenshoot](/images/openresty-13.jpg)

3. After everything is set, click **Save**!

![Screenshoot](/images/openresty-14.jpg)

## Configuring SSL

Setting up SSL is key to securing your users' data. You'll need to work with your DevOps team to get your SSL certificates in place, but here's how you can set things up in the OpenResty Edge dashboard.

### Manage SSL Certificates

In the **SSL** menu, you'll see a table to manage certificates. Here’s what you’ll typically need:

- Your SSL certificate (provided by a Certificate Authority).
- The private key associated with the certificate.
- Your domain, pointing to the correct gateway cluster.

Make sure all of these details are in order before proceeding.

![Screenshoot](/images/openresty-15.jpg)

![Screenshoot](/images/openresty-16.jpg)

![Screenshoot](/images/openresty-17.jpg)

### Check Gateway Clusters

Lastly, double-check the **Gateway Clusters** tab. Ensure the IP addresses listed here match the setup for your domain to avoid any connection issues.

![Screenshoot](/images/openresty-18.jpg)

---

You're almost done! Once you're satisfied with all the configurations, click the **Release** button. This will push your changes live, and your app will officially be up and running!

![Screenshoot](/images/openresty-19.jpg)

And there you have it! Your new application is now up and running, with all the necessary rules and configurations in place. If you have any questions or run into any issues, don’t hesitate to reach out. Here’s to smooth sailing and successful deployments!

Feel free to revisit this guide anytime you need a refresher or have new tweaks to make. Happy configuring!