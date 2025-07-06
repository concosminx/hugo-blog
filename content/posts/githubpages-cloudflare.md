---
date: '2025-07-03T12:18:07+03:00'
draft: false
title: 'Github Pages, Hugo and Cloudflare'
tags: ["linux", "hugo", "hosting", "cloud", "blog"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/pexels-viktortalashuk-2377295.jpg
    alt: 'Books'
---

### How to route your GitHub Pages site through Cloudflare on a Subdomain

Do you have a site hosted on **GitHub Pages**, like `https://usergithub.github.io/my-blog/`, and want to access it via a **custom subdomain** (e.g., `blog.yourdomain.com`) using **Cloudflare**? It's a great way to improve your performance, security, and have a cleaner URL. Here's how you can do it, step-by-step!

---

### Step 1: Prepare Your Domain in Cloudflare

If you haven't done this essential step already, you'll need to **add your main domain** (such as `yourdomain.com`) to Cloudflare. This process involves changing your domain's nameservers to those provided by Cloudflare [(see docs)](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/).

After you've changed the nameservers, it's important to be patient. DNS propagation can take from a few minutes to several hours. You can check the propagation status using online tools like `dnschecker.org`.

---

### Step 2: Configure the DNS Record in Cloudflare for Your Subdomain

This is where you connect your subdomain to GitHub Pages.

1.  Access your **Cloudflare** account and select your main domain.
2.  Go to the **DNS** section.
3.  You will add a `CNAME` type DNS record for your subdomain. This will direct traffic from your subdomain to the **base domain of GitHub Pages** (which is `usergithub.github.io`).

    * **Type:** `CNAME`
    * **Name:** Here you'll enter the **name of the subdomain** you want (e.g., `blog`). Your full subdomain will be `blog.yourdomain.com`.
    * **Target:** `usergithub.github.io`
    * **Proxy Status:** Make sure it's `Proxied` (the orange cloud should be active). This is crucial to benefit from all Cloudflare's advantages: CDN, security, performance optimizations, and a free SSL certificate.

---

### Step 3: Configure GitHub Pages for the Custom Domain

Now you need to tell GitHub that your site will be accessible via a custom domain.

1.  Navigate to your **repository** (`my-blog`) on GitHub: `https://github.com/usergithub/my-blog`.
2.  Click on **Settings** in the repository menu.
3.  In the left sidebar, click on **Pages**.
4.  In the "**Custom domain**" section, enter the **full subdomain** you want to use (e.g., `blog.yourdomain.com`).
5.  Click on **Save**.

    GitHub will verify the DNS records, and if everything is correct, it will configure your site to respond to the subdomain address. This process may take a few minutes.

---

### Step 4: Final Verification and Testing

* Wait a few minutes (or even up to an hour in rare cases) for all changes to propagate and be recognized globally.
* Try accessing your **subdomain** in your browser (e.g., `https://blog.yourdomain.com`). You should see your site's content.
* Check if the **SSL certificate** is active and if the site loads via HTTPS. The padlock icon should be present in your browser's address bar.

---

**A crucial point to remember:** For GitHub Pages sites hosted at the repository level, **you set the Cloudflare CNAME to point to `usergithub.github.io`**. It is not necessary to include the full repository path in the CNAME record. GitHub will automatically handle mapping the subdomain to the correct repository path once you've specified **"Custom domain"** in your repository's GitHub Pages settings.

---

**Optional - if you are using Hugo**

Update `baseURL` in Hugo Configuration

This step is **essential** for your Hugo site to function correctly under the new subdomain. If you don't do this, you might experience issues with links, images, and other resources.

1.  Open your Hugo project's configuration file. This is usually named `config.toml`, `config.yaml`, or `config.json` and is located at the root of your Hugo project.
2.  Find the `baseURL` parameter and update it with the **complete address of your new subdomain**, including `https://`.

    **Example in `config.toml`:**
    ```toml
    baseURL = "[https://blog.yourdomain.com/](https://blog.yourdomain.com/)"
    ```

    **Example in `config.yaml`:**
    ```yaml
    baseURL: "[https://blog.yourdomain.com/](https://blog.yourdomain.com/)"
    ```
3.  **Save the changes** in your GitHub repository. GitHub Pages will rebuild the site with the new configuration.