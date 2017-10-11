<!--
name = "Zen Code Life"
title = "Zen Code Life"
description = "Yongcheng's Technical Blog"
email = "yongcheng.chen@live.com"
author = "Yongcheng Chen"
categories_sort = {"about":1, "laravel":2, "angular":3, "magento":4, 'others':5}
-->
# A Static Site using Angular4 + AWS S3 + Lambda + Github

## Overview

This repository is about a solution of static website Angular4 App hosting on AWS S3 and content using Markdown file witch put on Github.

#### Supported Architectures

##### 1. Angular App
Please check sub folder Angular to view the source code.

##### 2. Github webhook Lambda
Please check Gitbot folder to view the source code

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
