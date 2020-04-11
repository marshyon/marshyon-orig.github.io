---
title: "Publishing a Hugo Site to Github Io"
date: 2020-04-10T17:18:18+01:00
draft: false
weight: 2
---

## Setting up your Github account

To publish to github.io for free, you need to have a current public Github account. Mine is already created under the username of **marshyon** so this is the name I used to publish this site under. A github.io account is available to all Github account users with their username as subdomain to github.io, for example: 

```
https://marshyon.github.io
```

so you will, when activated have access to your content from a similar url but with your username in place of **marshyon** of course.

To activate this functionality, create a new public repository in your Github account :

![create a new repo in Github](/images/new_repo.png)

so **Repository name** needs to be like the following :

```
<your_git_user_account_name>.github.io
```

so I created one like this :

```
marshyon.github.io
```

now I have my content available at 

```
https://marshyon.github.io
```

Any content ( html, css, javascript, jpg, png etc ... ) that gets commited to your newly created repository will then automatically appear at the <your_gith_user_name>.github.io address

There can be a time lag between your pushing your additions / changes to github of up to 20 minutes - accourding to Githubs own documentation so dont expect things to appear straight away, although they might, depenindg upon Githubs own current work loads. This is only fair in my view, after all, this is a free service so we cant really complain.

## Conigure Hugo for use in Github.io hosting

Create a new directory structure to host your new git account with the following :

```
.
├── hugo_site
└── README.md
```

where hugo_site is a directory that will hold your hugo site and README.md contains a description of your site and project

Initialise your git repository locally following the instructions given by github when you created the new repository.

Copy or move the hugo site you created with [getting started installation](/getting-started/installation) into the **hugo_site** directory.

Hugo uses a **config.toml** file to configure the creation of your static site so this needs to be changed to have the following values :

```
baseURL = "https://your_git_user_name.github.io"
publishDir = ".."
```

replacing **your_git_user_name** with your own git user name

ensure that all of your hugo pages have 

```
draft: false
```

in their meta data or have this line removed then change directory to your **hugo_site** directory and run :

```
hugo
```

You should find that your site code is generated in the directory above **hugo_site**.

Commit to git your additions and changes then push the git content to see the site available at github.io.

## Working with new content

the following can be used to run a dynamically loaded site on localhost port 1313 :

```
hugo serve -b http://localhost:1313
```

This over rides the github address now in config.toml and permits you to preview changes before again running hugo to update / git commit and push your generated content.
