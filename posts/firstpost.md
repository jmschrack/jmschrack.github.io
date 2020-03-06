---
title: Running eleventy on GitHub Pages for idiots.
description: A step by step 
date: 2020-03-05
tags:
  - 11ty eleventy
layout: layouts/post.njk
---
When it comes to text tutorials, I prefer the overly verbose tutorials. So I'm documenting my process of getting the Eleventy base blog. 

## What is this?
Eleventy is a "Static Site Generator" that lets you type of pages in simple markdown, and then generate all the html and CSS required.
GitHub Pages will host static HTML websites based on a git repo. 
TravisCI will auto pull your latest code, build it, and deploy for you.   
All 3 of these together make for a straight forward website.

## Get Node.js (Optional)
Eleventy uses JavaScript and runs on Node.js.  TravisCI will handle all of this for us, and so you don't have to install node.js yourself.
However, it's a good idea to do this so you can rapidly test changes on your local computer before pushing them live.

You can download an installer for Node.js from https://nodejs.org/en/download/. Let it do its thing.

## Get Eleventy
(From https://www.11ty.dev/ )
1. Open up Windows Powershell (or command prompt).
2. run "npm install -g @11ty/eleventy"  (without quotes)
3. Go to https://github.com/11ty/eleventy-base-blog and choose "Use This Template" button. Name your repo whatever you want, or leave the default name.
4. Go to https://travis-ci.org/getting_started , sign in with github. If you're feeling lazy, just activate Travis for all repositories.
5. On your GitHub repository, create a new branch called "gh-pages".
5. On GitHub, go to your profile Settings>Developer Settings>Personal Access Tokens, and create a new personal access token for TravisCI with the "Repo" permissions. **Copy the key!**
6. Go to TravisCI and navigate to your repository, and go to the settings for it. Create a new EnvironmentVariable called "GITHUB_TOKEN", paste the Access Token you copied in here and click "Add"
7. Edit \_data/metadata.json and replace some of the place holder data. We do this to do a fresh commit which will trigger our Travis build.
8. (Optionally) If you changed the name of your repo, you will need to edit your .travis.yml and change the "path prefix" to match your repo name.
9. Commit and push your changes now. Go back to TravisCI and watch your build begin. 
10. Once Travis finishes, go to your GitHub.com repo's settings, scroll down to find the "GitHub Pages" section. If everything worked, you'll be given a link! 

The first time, I got an "Unknown Tag" error. I don't know what caused it, but doing a second build on Jenkins fixed it.
``` text/2-3
// this is a command
function myCommand() {
	let counter = 0;
	counter++;
}

// Test with a line break above this line.
console.log('Test');
```
