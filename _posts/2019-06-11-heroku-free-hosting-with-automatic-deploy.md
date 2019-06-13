---
layout: post
title: "Free development hosting and automatic deploys from Github with Heroku"
author: bert
categories: [ tutorial ]
tags: [Heroku, hosting]
image: assets/images/2019-06-11-heroku-logo.png
description: "Learn how to host your applications for free using the Heroku free tier. Deploy automaticly when you push to github."
featured: false
hidden: false
---
Free quality hosting is hard to find. However, Heroku offers it for development. In this tutorial we'll explain
 how you can deploy your GitHub project online, for free, without any need for a credit card. This way you can test your application with
 a domain name and a server without any cost.
 
## What is Heroku?
Heroku is a cloud platform that lets anyone build, deliver, monitor and scale apps. The way this works is through _Dynos_. 
A dyno is a sort of container instance, with 512MB ram and a limited number of processes. The base tier is completely free,
and signing up can be done easily through github. No credit card or other payment method is required.  

![Heroku pricing]({{ site.baseurl }}/assets/images/2019-06-11-heroku-pricing.png)

### Free tier limitations
As can be seen in the image above, the free tier has some limits.

#### Worker limits
Only one web worker and one service worker. This is typically enough for normal projects. A web worker is a process or application which
reacts to HTTP requests. A service worker is an application which runs non-stop (for example, doing background processing).

#### Time limits
Accounts are given a base of 550 free dyno hours each month. In addition to these base hours, 
accounts which verify with a credit card will receive an additional 450 hours added to the monthly free dyno quota. 
This means you can receive a total of 1000 free dyno hours per month. 31 days correspond with 744 hours.
When you use all your free dyno hours for a given month, all free apps on your account will be forced to sleep for the 
rest of the month.
 
It is clear that a free account with 550 hours won't be able to run an entire month. However, if a web project doesn't
receive a request in 30 minutes, it will go to sleep. Sleeping dynos aren't counted towards your usage, meaning that a 
dyno which is only used one hour a day, will count up to 31 hours a month. 550 dyno hours correspond with about 
17h45m every day in a 31-day month. This should be more than enough for development purposes. You can even run multiple free dynos at once,
for example 3 free dynos for 6 hours every day. [The full details can be read here](https://devcenter.heroku.com/articles/free-dyno-hours).

## Deploying to heroku in 5 minutes
Deploying your github app to Heroku takes about 5 minutes. For this tutorial we will use our Google Assistant demo application, as it includes 
a Hello World welcome page for testing purposes. The repository can be found here: 
[https://github.com/trafiklab/google-assistant-demo/](https://github.com/trafiklab/google-assistant-demo/) 

### 1. Fork the repository to your own github account
Click the _fork_ button to _fork_ the repository to your own github account

![Forking a repository]({{ site.baseurl }}/assets/images/2019-06-11-github-fork.png)

### 2. Create a new Heroku app
Head to [dashboard.heroku.com/](https://dashboard.heroku.com/). Create an account, or log in if you have one already.
After logging in you get the option to create a new app. Choose a name and select the right region. 
The name will be used as the subdomain for your application later on.

![Creating a new app]({{ site.baseurl }}/assets/images/2019-06-11-heroku-new-app.png)

### 3. Link with github

Now your app has been created, you can link it with Github. Head over to the _Deploy_ tab, which looks like this.

![Linking with Github]({{ site.baseurl }}/assets/images/2019-06-11-heroku-app-deploy.png)

Select github as deployment method, and pick the repository to which you forked the demo app

![Configure automatic deploys]({{ site.baseurl }}/assets/images/2019-06-11-heroku-app-deploy-finished.png)

Your project will now deploy automatically when the master branch is updated! Woohoow!

### 4. Configure environment variables

Create a new config var. Use `APP_NAME` as name, and a value of your choosing as value. These environment variables will be set in the dyno,
and take effect immediately. They are typically used for API keys and other variables. These environment variables correspond with the export command in bash,

![Setting environment variables]({{ site.baseurl }}/assets/images/2019-06-11-heroku-settings.png)

### 5. Access your app

Click the "Open App" button in the top right corner. You will now be taken to the homepage of the our demo app, where you should see
"Hello world!", along with the value you configured in step 4. 

## For those who want more

### Where do my files go?
Dynos don't have permanent storage. This means that every written file will be deleted when the dyno restarts. If you want to store
files permanently, you will need to make use of external storage, for example Amazon S3 or database storage.

### How can I log without file storage?
First of all, your application is supposed to write to the standard output and error streams. This can be done by printing 
to the console, which can be done in Java by using the following code: 
```
System.err.println("Hello, logs!");
System.out.println("Hello, logs!");
```
PHP Laravel and Lumen projects by setting the environment variable `LOG_CHANNEL` to `errorlog`. 
For other examples, see [the Heroku docs](https://devcenter.heroku.com/articles/logging#writing-to-your-log).

Once your application is logging to the standard output and error streams, you can use [the Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)
 to access your application logs. After installation and configuration you can run `heroku logs` to see the last 100 lines of log output.
For more options, you can check [the full documentation](https://devcenter.heroku.com/articles/logging#log-retrieval).
 
### Configuring the deployment path

Specific deployment settings can be configured by adding a `Procfile` file to your repository. For example, to specify that our web dyno 
needs to use apache2 with the repos `public/` folder as root, we added the following file to our Trafiklab project.

```
# A Procfile defines settings for deploying an application to Heroku

# Lumen uses "public" as the web root.
web: vendor/bin/heroku-php-apache2 public/
```