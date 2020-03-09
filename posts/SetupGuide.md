---
title: An Idiot's Guide to running Eleventy on GitHub Pages.
description: Written as if you know almost nothing about any of this tech.
date: 2020-03-05
tags:
  - tutorial
  - blog
  - 11ty
  - eleventy
layout: layouts/post.njk
---
When it comes to text tutorials, I prefer them overly verbose. So I'm documenting my process of getting the Eleventy base blog for the next wandering programmer that might come by. 
I've used Continuous Integation before, but never TravisCI. I've used git and GitHub before, but never GitHub pages. And of course, I've never used eleventy before now.
The whole point of SSG is to enable you to just focus on the content of your website, and not the infrastructure. However, if you haven't used all of these components, guess what you're wasting time trying to figure out?
So an "Idiot's Guide" is to enable anyone (not necessarily developers!) to do the bare minimum infrastructure set up in order to focus on the content. 
The only assumption is that you know commit&push to a git repo.

## What is this?
Eleventy is a "Static Site Generator" that lets you type up pages in simple markdown, and then it generates all the html and CSS required.
GitHub Pages will host static HTML websites based on a git repo. 
TravisCI will auto pull your latest code, build it, and deploy for you.   
All 3 of these together make for a straight forward website pipeline.

## Get Node.js (Optional)
Eleventy uses JavaScript and runs on Node.js.  TravisCI will handle all of this for us, and so you don't have to install node.js yourself.
However, it's a good idea to do this so you can rapidly test changes on your local computer before pushing them live.

You can download an installer for Node.js from https://nodejs.org/en/download/. Let it do its thing.

## Get Eleventy
(From https://www.11ty.dev/ )
1. Open up Windows Powershell (or command prompt).
2. run "npm install -g @11ty/eleventy"  (without quotes)
3. Go to https://github.com/11ty/eleventy-base-blog and choose "Use This Template" button. Name your repo whatever you want, or leave the default name.
4. On your freshly created/cloned GitHub repository, create a new branch called "dev" (Or whatever you want, all your work will be done here.)  If you want to set this up for a specific repo instead of your "User page", call the branch "gh-pages", and skip to step 6.
5. On your github repo's settings, change the default branch to "dev" and rename your repo to _username_.github.io  

## Set up TravisCI to publish your GitHub repo
6. Go to https://travis-ci.org/getting_started , sign in with github. If you're feeling lazy, just activate Travis for all repositories.
7. On GitHub, go to your profile Settings>Developer Settings>Personal Access Tokens, and create a new personal access token for TravisCI with the "Repo" permissions. **Copy the key!**
8. Go to TravisCI and navigate to your repository, and go to the settings for it. Create a new EnvironmentVariable called "GITHUB_TOKEN", paste the Access Token you copied in here and click "Add"

## Publish your page for the first time
9. If you are not using the User page (i.e. you skipped step 5 above), just change the "pathprefix" in .travis.yml to point to your repo's name. Otherwise, edit your .travis.yml and change it to the following.:

```
language: node_js
node_js:
  - 8
before_script:
  - npm install @11ty/eleventy -g
script: eleventy --pathprefix="/"
deploy:
  local-dir: _site
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN  # Set in travis-ci.org dashboard, marked secure
  keep-history: true
  target_branch: master
  on:
    branch: dev
```


This tells TravisCI to pull from the dev branch, and deploy to the master branch. 
10. Commit and push your changes now. Go back to TravisCI and watch your build begin. 
11. Once Travis finishes, go to your GitHub.com repo's settings, scroll down to find the "GitHub Pages" section. If everything worked, you'll be given a link that points to _username_.github.io !


The first time, I got an "Unknown Tag" error. I don't know what caused it, but doing a second build on Jenkins fixed it.
