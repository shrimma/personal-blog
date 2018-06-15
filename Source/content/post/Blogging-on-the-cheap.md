---
title: "Blogging on the Cheap"
date: 2018-06-15
categories:
- blog
- azure
tags:
- blog
- azure
- blobstorage
- hugo
- budget
keywords:
- azure
autoThumbnailImage: false
thumbnailImagePosition: "left"
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city-750.jpg
metaAlignment: center
disqusIdentifier: db8f37f54a5f4200824feea54c4faf2e
---
First up, let's look at how this blog was created and hosted for less than the price of a single coffee.
<!--more-->

For hosting my blog I decided against using one of the traditional blog platforms such as Wordpress. Primarily as to run Wordpress (or similar) you need a web server, and a database, and etc, etc - all of this starts to add up in terms of cost and as I'm not expecting to get millions of views why should I be paying a significant amount of money to keep my little blog up and running.

Instead I opted to take the hipster approach and go for a static content provider, there are a number of options out there but I settled upon Hugo. My plan was to host the static content site in Azure Blob Storage and spend just pennies a month!

Below you'll find everything you need to know to get started with Hugo. Along with how to upload and host your static website in Blob storage. We even take it a step further and spin up Azure CDN to ensure speed and reliability for end users, no one waits for a 'slow' site in this day and age!

**Note you can do all of this with free Azure credits - trust me this isn't going to make a dent in your Azure wallet, as I said from the top we are talking PENNIES! (or cents if that's how your currency works)**

<!--toc-->
# Prerequisites
* Hugo - if you can then add hugo.exe to your environment PATH - it will help you out in the long run.
* Azure CLI v2 - we'll use this to upload our site to blob storage.

# Hugo
Firstly you'll need to download the latest version of hugo, if you haven't already. Did you skip the prerequisites section? Shame on you!

Let's use hugo to create our site, fire up command prompt and follow along with these commands:
```
hugo new site myblog
```

We want our new blog to be shiney and preferablly responsive on different size screens/devices so we need to pick a theme which takes care of all this hard work for us. Personally I settled on [tranquilpeak](https://github.com/kakawait/hugo-tranquilpeak-theme) for my blog but it requires a bit more configuration out of the box so for the purposes of keeping this post simple let's use [ananke](https://github.com/budparr/gohugo-theme-ananke). It's important to note that there are a wide variety of themes available, so pick one that works for you.

You'll need to clone the selected themes repo into the themes subfolder of your new site.
```
cd myblog/themes
git clone https://github.com/budparr/gohugo-theme-ananke.git
```

Once we've cloned the theme we need to update our config to use it. Head back to the root directory of mysite and edit config.toml in your text editor of choice. Add a 'theme' parameter with the value set to whichever theme you just cloned.
```
theme = "gohugo-theme-ananke"
```

Our new blog is going to need some content so let's create our first post. *You should run this command from your sites root directory so run the 'cd ..' as required.*
```
cd ..
hugo new post/my-first-post.md
```

Once you've created your dummy post then fire up your markdown editor of choice, I'm using haroopad, and add a nice warming welcome message to any potential visitors.
```
Hello World!
```

Let's take an early look at our handiwork. Helpfully hugo includes a lightweight server to make this really simple.
```
hugo server
```

Leaving the command window open, fire up a browser and navigate to [localhost:1313](http://localhost:1313).
![First look](/img/firstlook.png)
Tada, you've created your first static website! Take a moment to pat yourself on the back and then we'll plough on and get our site hosted and served from Azure.

*If you like you can spend some time now to customize the theme and thus your blog to your hearts desire, just refer to your selected themes documentation for guidance around this. Hell start adding content if you want. It really is up to you.*

# Blob Storage
Now we come to our hosting mechanism, in this case we are avoiding the tradional IIS/webserver and relying on plain blob storage. Blob storage fits with hugo perfectly in my mind and you'll see why shortly.

Create a new Storage account in Azure, you can use the portal if you like but we'll perform the same with Azure CLI.

Firstly we need to login. Type the following and follow the prompts.
```
az login
```

Now we'll create a new resource group and storage account
```
//creates a new resource group
az group create --location westeurope --name MyNewBlog
//creates the blob storage account
az storage account create --name jsblogstorage1 --resource-group MyNewBlog --location westeurope --sku Standard_RAGRS --kind BlobStorage --access-tier Hot --https-only true
```

Once our storage account has been deployed we can create a container to hold our blog but first off we need to grab our storage account connection string
```
az storage account show-connection-string --resource-group MyNewBlog --name jsblogstorage1
```

Take a note of connection string as you'll need it for the next steps. Let's create our container.
```
az storage container create --name testblog --public-access container --connection-string YourConnectionStringHere
```

Let's take a second to cover one very important thing. You may have noticed when you opened up config.toml earlier a 'baseurl' setting with the value 'example.org' - this becomes very important when hosting your static site as it is the glue for all the css, javascript, hyperlinks, etc of your site. If you want to be able to view your site, without it looking very rough around the edges, you'll want to set this value to your blob containers URL (at least for now). The pattern for your containers url will be: [https://YourStorageAccountName.blob.core.windows.net/YourContainerName]()

Mine is [https://**jsblogstorage1**.blob.core.windows.net/**testblog**]()

Edit config.toml and set the baseURL to your containers URL.
```
baseURL = "https://jsblogstorage1.blob.core.windows.net/testblog/"
```

Let's build our hugo site. Simply enter hugo into your command line without any other parameters/arguments.
```
hugo
```

To upload our site to blob storage we'll again use Azure CLI v2. We've come so far with this why not stick with the command prompt vibe in this post.
```
az storage blob upload-batch --source public --destination testblog --connection-string YourConnectionStringHere
```

And simple as that our blog is now available to the world wide web! Don't believe me? Paste your base url plus index.html into another tab and have a look - Mine is [https://**jsblogstorage1**.blob.core.windows.net/**testblog**/index.html]()

You can stop here if you like, after all our blog site is "live", but we can do better than this! First off we don't want people to have to remember to include index.html an the end of every URL (seriously what kind of masochist expects that of users) and secondly those blob URLs are very forgettable.

# CDN
Using Azure CDN we can create rules that will mean our users don't see any unexpected 404 errors when they reach [www.ourblog.com]() rather than [www.ourblog.com/**index.html**](). Also we get improved response time globally. Win win!




# HTTPS, Domain, Certificate
Now the cherry on top, you can use Azure CDN in combination with any custom domain that you happen to own to say good bye to the forgetable and untypable blob storage URLs and hello to the simple and easy to remember [mysuperduperawesomeblog.com]().

# Finishing Up
With all posts I will endeavor to include any source code on Github. This post is actually an exception to that rule (rules are made to be broken right?) as rather than uploading a skeleton blog which wouldn't give you any value you can instead find my live blog code. Head on over to the repo and check it out if you like, you may even get a sneak peak at what's coming next...

# Next Time
Let's get our DevOps hats on! We want deployments to be reliable and repeatable so we'll look at using ARM templates to create our azure resources. Plus we'll look at how we can harness the power of VSTS build/release pipelines to create a continuous integration and continuous deployment pipeline that will do all the leg work for us.

As always leave any comments below the line :)


