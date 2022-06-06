---
title: "How did I build this website?"
date: 2021-04-12T00:05:06-04:00
author: "ozan ocak"
tags: ["hugo", "website", "git"]
---

Hugo is a static HTML and CSS website generator written in Go. It is optimized for speed, ease of use, and configurability. Hugo takes a directory with content and templates and renders them into a full HTML website.

I chose Hugo to this blog and store it in github as a static page. First I install hugo with brew then create 2 seperate github repository; one for all the source codes and the other for staic files in public folder.

```bash
brew  install hugo

hugo new site my_blog
```

After Hugo set up initial folders and files, I wanted fetch one of hugo theme with git submodule. Git submodules allow you to keep a git repository as a subdirectory of another git repository.

```bash
git init .

cd my_blog/themes

git submodule add https://github.com/WingLim/hugo-tania
```

Hugo uses the config.toml, config.yaml, or config.json as the default site config file. I prefred to use toml which is another file format for configuration files.

```bash
mv config.toml config.yaml

vim config.yaml

baseURL: "https://bitsandbytes.github.io"
languageCode: "en-us"
title: "fascinating journey of bits and bytes"
theme: "hugo-tania"
```

Now I can start deamon process to serve the website in localhost thanks to golang.

```bash
hugo server

hugo new post/hello.md

git clone ozanocak.github.io.git

git checkout -b main (now it is master branch)
```

Checking out main branch in cloned git repository makes it master page so I don't need to explicitly add remote connection to the repository.

```bash
touch readme.md
git add .
git commit -m "readme.md"
push -u origin main

git submodule  add -b main ozanocak.github.io.git public

hugo -t hugo-tania
```

when I cd public & git remote -v , I see origin is the host directory because I branched&pushed it already to the repository.Finally I can add changes git.

When I need to update the website, I need to go to directory where yaml config file located to generate updated binary files and push the changes to github page.

```bash
hugo
git status
git add .
git commit -m "some changes"
git push -u origin main
```

To see the changes in local host we need to use -D flag because the files as default are draft which are not appear in localhost.

```bash
hugo new about.md
hugo server -D
```
