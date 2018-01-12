<!-- 
Categories = ["Development", "Angular", "AWS"]
title = "A Static Site using Angular4 + AWS S3 + Lambda + Github"
Description = "A Static Site using Angular4 + AWS S3 + Lambda + Github"
Tags = ["Development", "Angular 2/4"]
date = "2018-01-12T10:38:31+11:00"
-->

## A Static Site using Angular4 + AWS S3 + Lambda + Github

### Overview

This [github repo](https://github.com/yongchengchen/angular_s3_github_staticwebsite) is a solution of static website Angular4 App engine on AWS S3 and using Markdown file content witch managed by Github.

### Supported Architectures

#### 1. Angular App

It's in the folder src. It includes three sub-folders:

##### 1) ngsuit

"ngsuit" is a framework which can boost your Angular development.
It has built a basic Angular app entry, you can add new features by just adding new module folder such as Blog and RootLayout, and configure it in your module's assemble.ts
It also built some basic components, services and pipes 
such as [MQService](http://zencodelife.me/article/angular/angular2componentcommunication.md), [GobalVariableService](http://zencodelife.me/article/angular/angular2globalvariableservice.md), LoaderLayer component, asyncprocess component.

And "ngsuit" has built some hooks to support your development.

So you just run command "npm run start" or "npm run build", it will automaticly link your module, and assemble these module's configurations, and build your app.

##### 2) RootLayout

It's a layout module with some layout component such as header, sidebar.

##### 3) Blog

It includes home page component, article component and aws blog data content service.
Aws blog data content service it pull content summary data from aws. Which content summary data are created/updated by "gitbot"(an AWS Lambda function which parse my github blog content repo).

##### 4) Build

Need to go to ngsuit folder to build this app.

```shell
cd path_to_ngsuit
npm install //if npm modules not installed
npm run build
```

##### 2. Github webhook Lambda (gitbot)

This project is an AWS Lambda function project using node.js and webpack to provide a github webhook.

When github repo updates it will call this webhook, and this webhook to parse latest github repo's changes, and then update this repo's summarry to AWS s3.

Then the static Angular blog app can load the latest data from github.

###### Build
```shell
npm install //if npm modules not installed

cd gitbot
npm run build
```

Check dist/index.js and deploy it to your AWS Lambda


**Complete documentation is available at [Documentation][].**

## Some rules

#### 1. Github repository
We analyse github repository from it's first level folder "content". So far we only support one levele category.

So the normal repository directory structure should looks like:
```
    repository
       |-------README.md
       |
       |-------content
                  |--------about.md
                  |--------category1
                  |            |--------file1.md
                  |            |--------file2.md
                  |            
                  |---------category2
                               |--------file1.md
                               |--------file2.md
``` 

#### 2. Markdown file

It relies on Markdown files with front matter for metadata, you should put metadata at the begining of your file.
And we support these keys: Categories, Description, Tags, date, title

* Please use only double quote in the metadata field.

```html
<!--
Categories = ["Development", "Others"]
Description = "Some time your will not want your company's code goes to public project, but you want composer to manage packages.
Here's an example for how to let composer use private packages, and put these packages to customize folder."
Tags = ["Development", "Composer"]
date = "2016-11-25T21:47:31-08:00"
title = "Composer use private git packages"
image = ""
-->
```

#### 3. README.md
README.md also support metadata extension. But keys are different.
It stores site level configuration details.
```html
<!--
name = "Your website name"
title = "Your website title"
description = "Your website description"
email = "put your email address here"
author = "Put your name here"
-->
```

## Installation

## Contribution

to be continue