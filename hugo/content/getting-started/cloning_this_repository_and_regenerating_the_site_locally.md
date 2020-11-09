---
title: "Cloning this site and regenerating locally on your own machine"
date: 2020-11-09T13:00:00+01:00
draft: false
weight: 1
---

## Prerequisites

to download the site and push it back to the same or other repositories you need Git. 

If you dont already have Git, it can be downloaded from https://git-scm.com/downloads

to rebuild or preiview the site you need `hugo` in your path. It is a single executable with no installer, just copy the binary somewhere like (for windows) 'c:\program files\<some directory of your choice> and add it to your system path in `System Properties - Environment_variables` if it is not already or (for Linux / Mac) `/usr/local/bin`.

Hugo can be downloaded from [hugo releases](https://github.com/gohugoio/hugo/releases)

## Copy / clone the site

This site is hosted at [https://marshyon.github.io/](https://marshyon.github.io/).

Clone it to somewhere appropriate on your system with :

```
git clone https://github.com/marshyon/marshyon.github.io.git
cd marshyon.github.io.git/hugo/themes/hugo-theme-learn
git submodule init
git submodule update
cd ../../
```

## Edit content

open the directory and edit the files in 

```
hugo/content
```

## Previewing changes 

from within the `hugo/content` directory run `hugo serve -b http://localhost:1313` to serve the content locally 

## Rebuild the site post to making changes

To regenerate the site with new content in `hugo/content` from a command line window and within the `hugo` directory run :

```
hugo
```

I you don't want to add hugo to your path, it can be run from the place you downloaded and uncompressed it to, for example in windows :

```
C:\Users\<your user>\Downloads\hugo_0.78.0_Windows-64bit\hugo.exe
```

## Publishing changes back to Github

If you have access to push back to the marshyon account in which this site is hosted, just push changes back with 

```
git push
```

### Changing the site to be used in your own git repo

If your wanting to use this repository for your own site, replace with your content in `hugo/content` and edit `config.toml` accordingly to suit your intended site name

in `.git/content' ( at the root folder of this repository directory ) edit the `config` file and change the section 

```
[remote "origin"]
        url = https://<your remote git repository here>.git
```

to have the url of your git repository and then push to it with `git push`
