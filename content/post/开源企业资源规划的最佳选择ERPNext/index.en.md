---
title: New Era ERP System-ERPNext
description: This article is copied from GitHub.
date: 2026-02-02T18:34:25-08:00
draft: false
categories:
- APP
- GitHub
tags:
- ERPNext
---

# New Era ERP System-ERPNext

## Introduction

[100% Open-Source ERP system to help you run your business.](https://github.com/frappe/erpnext)

### Motivation

Running a business is a complex task - handling invoices, tracking stock, managing personnel and even more ad-hoc activities. In a market where software is sold separately to manage each of these tasks, ERPNext does all of the above and more, for free.

### Key Features

- **Accounting**: All the tools you need to manage cash flow in one place, right from recording transactions to summarizing and analyzing financial reports.
- **Order Management**: Track inventory levels, replenish stock, and manage sales orders, customers, suppliers, shipments, deliverables, and order fulfillment.
- **Manufacturing**: Simplifies the production cycle, helps track material consumption, exhibits capacity planning, handles subcontracting, and more!
- **Asset Management**: From purchase to perishment, IT infrastructure to equipment. Cover every branch of your organization, all in one centralized system.
- **Projects**: Delivery both internal and external Projects on time, budget and Profitability. Track tasks, timesheets, and issues by project.

<details open>


<summary>More</summary>
	<img src="https://erpnext.com/files/v16_bom.png"/>
	<img src="https://erpnext.com/files/v16_stock_summary.png"/>
	<img src="https://erpnext.com/files/v16_job_card.png"/>
	<img src="https://erpnext.com/files/v16_tasks.png"/>
</details>

### Under the Hood

- [**Frappe Framework**](https://github.com/frappe/frappe): A full-stack web application framework written in Python and Javascript. The framework provides a robust foundation for building web applications, including a database abstraction layer, user authentication, and a REST API.

- [**Frappe UI**](https://github.com/frappe/frappe-ui): A Vue-based UI library, to provide a modern user interface. The Frappe UI library provides a variety of components that can be used to build single-page applications on top of the Frappe Framework.

## Production Setup

### Managed Hosting

You can try [Frappe Cloud](https://frappecloud.com), a simple, user-friendly and sophisticated [open-source](https://github.com/frappe/press) platform to host Frappe applications with peace of mind.

It takes care of installation, setup, upgrades, monitoring, maintenance and support of your Frappe deployments. It is a fully featured developer platform with an ability to manage and control multiple Frappe deployments.

<div>
	<a href="https://erpnext-demo.frappe.cloud/app/home" target="_blank">
		<picture>
			<source media="(prefers-color-scheme: dark)" srcset="https://frappe.io/files/try-on-fc-white.png">
			<img src="https://frappe.io/files/try-on-fc-black.png" alt="Try on Frappe Cloud" height="28" />
		</picture>
	</a>
</div>




### Self-Hosted

#### Docker

Prerequisites: docker, docker-compose, git. Refer [Docker Documentation](https://docs.docker.com) for more details on Docker setup.

Run following commands:

```
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
docker compose -f pwd.yml up -d
```

After a couple of minutes, site should be accessible on your localhost port: 8080. Use below default login credentials to access the site.

- Username: Administrator
- Password: admin

See [Frappe Docker](https://github.com/frappe/frappe_docker?tab=readme-ov-file#to-run-on-arm64-architecture-follow-this-instructions) for ARM based docker setup.


## Development Setup

### Manual Install

The Easy Way: our install script for bench will install all dependencies (e.g. MariaDB). See https://github.com/frappe/bench for more details.

New passwords will be created for the ERPNext "Administrator" user, the MariaDB root user, and the frappe user (the script displays the passwords and saves them to ~/frappe_passwords.txt).


### Local

To setup the repository locally follow the steps mentioned below:

1. Setup bench by following the [Installation Steps](https://frappeframework.com/docs/user/en/installation) and start the server

   ```
   bench start
   ```

2. In a separate terminal window, run the following commands:

   ```
   # Create a new site
   bench new-site erpnext.localhost
   ```

3. Get the ERPNext app and install it

   ```
   # Get the ERPNext app
   bench get-app https://github.com/frappe/erpnext
   
   # Install the app
   bench --site erpnext.localhost install-app erpnext
   ```

4. Open the URL `http://erpnext.localhost:8000/app` in your browser, you should see the app running

## Learning and community

1. [Frappe School](https://school.frappe.io) - Learn Frappe Framework and ERPNext from the various courses by the maintainers or from the community.
2. [Official documentation](https://docs.erpnext.com/) - Extensive documentation for ERPNext.
3. [Discussion Forum](https://discuss.frappe.io/c/erpnext/6) - Engage with community of ERPNext users and service providers.
4. [Telegram Group](https://erpnext_public.t.me) - Get instant help from huge community of users.











