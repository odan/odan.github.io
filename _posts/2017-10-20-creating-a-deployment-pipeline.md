---
title: Creating a deployment pipeline
layout: post
comments: true
published: false
description: 
keywords: 
---

* [Release management process](#release-management-process)
* [Environment Configuration](#environment-configuration)
* [Deployment](#deployment)

## Release management process

This topic discusses the environments used in the release management process for a WebApp solution. 
As with any enterprise software solution, you should follow established software 
release management guidelines when you develop and release a solution. 
This process should include the following distinct stages:

* Development
* Testing
* Staging
* Production

Ideally, you should complete each stage in the release management process in a discrete environment, 
separate from the other environments. Realistically, you may have to combine one or more of 
the environments due to hardware, time, or other resource constraints. 
At a bare minimum you should separate the production environment from the other environments.

### Development

The projects are created in the development environment. 
Typically, developers should have their own development computer (physical or virtual) 
with the necessary software installed. You should install the following software on the 
computers used in the development environment:

* XAMPP (with Xdebug)
* PhpStorm
* Different Browsers (Chrome, FF, IE)
* SQLyog or MySql Workbench, [SQLyog](https://github.com/webyog/sqlyog-community) or phpmyadmin
* Notepad++, Putty, WinSCP, WinMerge, PoEdit
* Composer, PHPUnit

### Testing

Unit testing can be completed in a Virtual Server environment. 
You should, however, conduct your performance testing in a physical environment with 
hardware and software that is identical to the production environment.

The testing environment should be used as a “internal testing” Environment

## Staging

You typically use the staging environment to “unit test” the actual deployment of the solution. 
The software installed in the staging environment should closely match the software installed 
in the production environment. It may, however, be acceptable to use computers running 
Virtual Server in the staging environment since this environment is not to be used 
for measuring performance.

The testing environment should be used as a “external testing” environment

### Production

The production environment is the “live” environment that will host the running solution. 
The production environment is the final endpoint in the release management process and 
should only host applications that have previously undergone development, unit testing,
load testing, and staging in the other environments. Thorough unit testing, load testing, 
and staging beforehand will help ensure maximum performance and uptime for the 
application in the production environment.

## Environment Configuration

We need to persist data across deployments, and don't want to commit these files to Git.

You should store all sensitive information in 'env.php' and add the file to your .gitignore, 
so that you don't accidentally commit to source control.

Rename the file 'env.example.php' to 'env.php'

You can store the `env.php` file in your storage directory.

### Usage

The `env.php` file is generally kept out of version control since it can contain sensitive 
API keys and passwords.

A separate ´env.example.php´ file is created with all the required environment variables 
defined except for the sensitive ones, which are either user-supplied for their own 
development environments or are communicated elsewhere to project collaborators.

The project collaborators then independently copy the env.example.php file to a 
local env.php and ensure all the settings are correct for their local environment, 
filling in the secret keys or providing their own values when necessary.

In this usage, the `config/env.php` file should be added to the project's `.gitignore` 
file so that it will never be committed by collaborators.

This usage ensures that no sensitive passwords or API keys will ever be in the 
version control history so there is less risk of a security breach, and production 
values will never have to be shared with all project collaborators.

### Best practice

In testing, staging and production environment you should place the env.php outside 
the application root directory. 
By placing the env.php outside the app, you are protecting the configuration from 
being accidentally overwritten. This makes it possible to update to a new version 
without worrying about the environment specific configuration.

By default, the application is looking for a `env.php` outside the app. 
If the file not exists the application is looking for a file in ´config/env.php´.

Pros:

* No need to change the webserver settings or maintaining .htaccess, vhosts etc.
* No special permissions required
* No sensitive information's are stored in environment variables
* More secure and easy to automate deployments

## Other solution

Using `.env` files with phpdotenv is **not recommended** because of security reasons.

Read more: [Environment Variables Considered Harmful for Your Secrets](http://movingfast.io/articles/environment-variables-considered-harmful/)

## Continuously delivering

This is the pipleline:

### Deployment pipeline

#### COMMIT > BUILD > TEST >| DEPLOY

### Generate an artifact

1. Check out master branch
2. Create a `env.php` file for each environment
 + Only one artifact for all environments (prod, staging, test, etc).
 + The `env.php` file contains the secret environment parameters like database credentials (DSN) and is not part of the artifact.
3. Fetch the PHP dependencies (composer install/update)
4. Fetch the JS dependencies (webpack) [optional]
5. Generate the assets (LESS, SASS) [optional]
6. Run the unit tests
7. Generate the artifact (zip file)
 + The artifact is a bundle of your release with php, vendor, js and css files
 + Build an artifact (zip) with Apache Ant (build.xml), Jenkins, within your IDE or command line
 + Discard unnececarry folders and file from the artifact (.git, tests/, sass, less)

## Deployment

Deploy the new release (with ansible deploy.yml) or with a php script like ([deploy.php](https://gist.github.com/odan/f683326e65ff44c2788d))

Steps (manual or by script):

* Upload artifact (ZIP) via SSH or fetch from Amazon S3
* Uncompress zip file (unzip release_2015-06-28_22-12-00.zip)
* Apply right permissions to project root (chmod -R 0755 tmp/)
* Make a folder switch
* Rename the current 'htdocs' folder (mv htdocs/ htdocs_2015-10-12 14-12-00/)
* Rename the 'release' folder to 'htdocs' (mv release_2015-06-28_22-12-00/ htdocs/)
 * Start/restart services [optional]
 * Run (smoke) test. [optional]

## Books

[Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation](http://www.amazon.de/dp/B003YMNVC0/ref=pe_386171_48771151_TE_M1T1DP)

## Videos

[Javier Lopez - Continuously delivering PHP projects](https://www.youtube.com/watch?v=HOwPJg-F4h0)