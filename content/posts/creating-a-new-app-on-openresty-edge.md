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

Welcome aboard! If you’re gearing up to launch a new application on **OpenResty Edge**, you’re in the right place. Whether you’re a seasoned pro or just starting out, I’ll walk you through every step of the way with a dash of fun. Grab your favorite drink, settle in, and let’s dive into this setup adventure!

---

## What is OpenResty Edge?

OpenResty Edge is a powerful platform designed to manage and deliver your applications at the edge, optimizing performance, scalability, and security. It allows you to deploy and configure applications in a fast, flexible, and efficient way, using advanced features like traffic routing, custom rules, and SSL management. With OpenResty Edge, you can easily manage traffic, enhance security, and ensure smooth user experiences for your applications at scale.

---

## Let's Get Started!

### Step 1: Creating Your New Application

Let’s get that new app live and kicking! Here’s how to create your first application on OpenResty Edge.

1. **Login to Your OpenResty Edge Dashboard**

    - Head over to the OpenResty Edge dashboard and log in using your credentials.
    
        ![Screenshoot](/images/openresty-1.jpg)

2. **Navigate to the Applications Section**

    - Once you’re logged in, click on **Applications** from the top menu.
    - Inside the **Applications** section, click on the **Create New App** button to start the process.

        ![Screenshoot](/images/openresty-2.jpg)

3. **Fill Out the Application Details**

    - Here’s a breakdown of the fields you’ll encounter:
        - **Domains**: Enter your domain name here (e.g., `tomi-example.co.id`).
        - **HTTP Port**: You can leave this blank for now.
        - **HTTPS Port**: Set to `443` for secure traffic.
        - **Label**: Give your app a memorable label, like `Create a New Example App`.

        ![Screenshoot](/images/openresty-3.jpg)

    - Once you’ve filled everything out, click **Save!**

---

### Step 2: Configuring Page Rules

Now, let’s configure some important rules to control how traffic behaves for your application.

1. **Custom Edge Language Rules at the Beginning of This Page**

    - To block specific IPs and log alerts, use the following rule:

        {{< highlight bash >}}

        client-addr !~~ any (192.168.100.1) =>
            errlog(level: "alert", "SEC-00- Unauthorized IP!", foreign-call(module: "headers-log", func: "go") ~ req-body()),
            exit(444);

        {{< /highlight >}}

        ![Screenshoot](/images/openresty-4.jpg)

    - This rule does two things:
        - Blocks traffic from the IP `192.168.100.1`.
        - Logs an alert with the message "Unauthorized IP!".

    - After adding your custom rules, click **Save**.

2. **Create New Rules**

    Let’s move on to setting up some common rules you'll probably need.

    ![Screenshoot](/images/openresty-10.jpg)

    **Redirect HTTP to HTTPS**

    To enhance security, redirect all HTTP traffic to HTTPS:

    - Click **New Rule** in the Page Rules section.
    
    - Set up the condition:
        - **When** → Add a new condition:
        - **Variable** → **Scheme**
        - **Operator** → **String =**
        - **Value** → **http**

        ![Screenshoot](/images/openresty-5.jpg)

    - Add the action:
        - **Action Type** → **Redirect**
        - Set the following fields:
        - **URI** → Choose **current**.
        - **Scheme** → Set this to **https**.
        - **Status** Code → Set to **302 Moved Temporarily**.

            ![Screenshoot](/images/openresty-6.jpg)


    - This will ensure all users trying to access your site via HTTP will be securely redirected to HTTPS.

    **Set Proxy Header**

    If you need specific headers to be passed to the backend server:

    - Click **New Rule** again.
    - In the **When** section, add a new condition for **Host**:
        - **Variable** → **Host**
        - **Value** → Your domain (e.g., `tomi-example.co.id`).

            ![Screenshoot](/images/openresty-7.jpg)

    - Add the following actions:
        - **Set Proxy Header**: Set the **Host** header with your domain name.
        - **Append Proxy Header**: Append the **X-Forwarded-For** header with the client address.

            ![Screenshoot](/images/openresty-8.jpg)

    - Under **Proxy** settings, turn **Proxy** On and select the upstream server you created earlier.

        ![Screenshoot](/images/openresty-9.jpg)

    **Custom Edge Language Rules at the End of This Page**
    
    - Wanna speed up your site? Caching static files like JavaScript, CSS, and images is a great way to do that. Here's the rule you need to add:


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

    - Here's the breakdown:
        - JavaScript and CSS files: Cached for 7 days.
        - Other files like images: Cached for 365 days.
        - Cache-Control Header: Tells browsers to keep these files for the specified period, which helps make repeat visits faster.
        - Save the Cache Rules

    - Once you’ve added these caching rules, be sure to click **Save** to lock it in place.

---

### Step 3: Configuring Upstreams

Next, let’s set up the backend server where your app will forward requests.

1. **Go to the Upstreams Menu**
    - Click on **Upstreams** from your dashboard.

2. **Add a New Upstream**
    - Enter the following details:
        - **Upstream Name**: e.g., `example-backend`.
        - **Protocol**: Select **HTTPS**.
        - **Host**: Enter your backend server domain or IP (e.g., `tomi-backend.int`).
        - **Port**: Set to **443**.
        - **Weight**: Set to **1**.
        - **Enabled**: Switch this **On**.

            ![Screenshoot](/images/openresty-12.jpg)

            ![Screenshoot](/images/openresty-13.jpg)

3. **Save the Upstream**

    - Click **Save** to apply the settings.

    <!-- ![Screenshoot](/images/openresty-14.jpg) -->

### Step 4: Configuring SSL

It’s time to secure your app with SSL! Here’s how to configure SSL certificates:

1. **Manage SSL Certificates**

    - In the **SSL** menu, you'll see a table to manage certificates. Make sure all of these details are in order before proceeding:

        ![Screenshoot](/images/openresty-15.jpg)

        ![Screenshoot](/images/openresty-16.jpg)

        ![Screenshoot](/images/openresty-17.jpg)

2. **Check Gateway Clusters**

    - Double-check that the IP addresses in the Gateway Clusters tab match your domain's setup to avoid connection issues.

        ![Screenshoot](/images/openresty-18.jpg)

---

### Step 5: Go Live!

Once all configurations are in place, hit the **Release** button to make your application live.


![Screenshoot](/images/openresty-19.jpg)

That’s it! Your app is now fully configured and live on OpenResty Edge. If you need help or have questions, feel free to reach out. Enjoy your new app setup!

---

## Conclusion
And that's a wrap! You've successfully created your application, set up page rules, caching, and all the necessary configurations to get your app up and running on OpenResty Edge. With these steps in place, your app will be secure, fast, and ready for the world to see.

If you ever need to tweak or update things, just refer back to this guide. Happy deploying, and enjoy your smooth sailing with OpenResty Edge!