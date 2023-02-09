---
title: "Ngrock Secure Tunnels to Localhost"
date: 2023-02-05T23:18:20+03:00
lastmod: 2023-02-05T23:18:20+03:00
weight: 3
draft: true
description: This lecture notes introduces the use of secure tunnels to your localhost and dev environment. You will learn how to use `ngrock` to expose an application running on a specific port on your local server (localhost) to the public Internet.
---

As you develop a software in your local environment, you may run into a scenario where you want to share a demo running in your local development environment with someone over the public Internet. Typically this scenario can be achieved by deploying the application into a staging environment that can be accessed internally for development and testing purposes. However, this may not be an option when working with a small team that does not have enough resources to provision a staging/testing environment or you if you want to quickly share a demo running locally.

[ngrok](https://ngrok.com) is a tool that establishes a secure TCP tunnel (outbound connection) between your local resources and the a server owned by ngrok over the public Internet without changing your network configurations, opening ports on your router, or having a publicly routable IP address.

While ngrok is not the only tool that provides tunneling services, it's one of the easiest and gained a popularity after being released as an open source project for version 1.x, which is no longer developed as an open-source project in favor of the commercial 2.x release. There are many TCP tunneling alternatives to ngrok including, [localtunnel](https://github.com/localtunnel/localtunnel) and [CloudFlare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/), and many others listed here in [this curated list of tunneling tools and services](https://github.com/anderspitman/awesome-tunneling).


## Exposing a database service (PostgreSQL) running on localhost to the public Internet

We are going to see how to use a tool like `ngrok` to expose a local database server (PostgreSQL) to the public Internet over secure tunnels.

### Step 1: Run your PostgrSQL locally

You may run your Postgres instance locally from the command line or using a GUI tool like pgAdmin. Postgres runs by default on localhost port 5432.

### Step 2: Create an account on ngrok.com and install the ngrok agent
- Create an account on ngrok.com and connect the local agent to your account.
- [Sign in using the](https://dashboard.ngrok.com/get-started/your-authtoken)

### Step 3: Connect your agent to your ngrok account



