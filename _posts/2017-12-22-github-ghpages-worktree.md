---
layout: post
title: GitHub Pages with gh-pages branch 
tags:
    - Git
    - Jekyll
---

Today, I wanted to create an website for my new tool [DocumentDb Explorer](https://www.bruttin.com/DocumentDbExplorer/){:target="_blank"}. As the source code is hosted on [GitHub](https://github.com/sachabruttin/DocumentDbExplorer){:target="_blank"} I wanted to use GitHub Pages to host it also. 

### GitHub Pages

With [GitHub Pages](https://pages.github.com/){:target="_blank"} you can host freely your website from your GitHub repository. You just need to push your static content or Jekyll website to a branch called ```gh-pages``` and they'll publish it on ```<your-username>.github.io/<repo-name>``` a few seconds later.

### Create your gh-pages branch

That's nice, but I started with an existing repo and didn't wanted to mix my work on the app with the website. The idea is to create an _orphan_ branch:

```powershell
git checkout --orphan gh-pages
```

When you create an _orphan_ branch, git creates a new branch without any parent commits. You can add anything you want to that branch and it's totally separate from the main history, but stored on the same .git directory.

I then cleaned my directory and added the two needed pieces to have a nice website, a Jekyll config file (_config.yml) and the index.md file that will be renderer as my home page.

The _config.yml file is very simple:

```yml
theme: jekyll-theme-cayman
title: DocumentDb Explorer
show_downloads: "true"
google_analytics: <YOUR-GOOGLE-KEY>
```

It's now time to push the new branch to GitHub.

```powershell
git push -u origin gh-pages
```

The _gh-pages_ branch is now pushed to GitHub and the site is available. For later changes ```git push``` will be enough.

### Worktree

Good, now I have two branches and I have to ```git checkout <branch>``` everytime I want to make change to the website or the application. 

My goal is to have a both branches on the same local directory and have each directory linked to his own remote branch. To achieve that, I need to use a feature of git called [worktree](https://git-scm.com/docs/git-worktree){:target="_blank"} that let me manage multiple working trees attached to the same repository.

To create my directory structure correctly I need to execute these commands:

```powershell
mkdir <my-root-folder>
cd <my-root-folder>

git clone https://github.com/<user>/<repo> work # this create a work folder where I will have my app
cd work
git checkout gh-pages # pull down remote branch and make it a local one
git checkout master # switch back to the branch matching the directory tree

mkdir ..\gh-pages # create the gh-pages folder as the same level the work folder
git worktree add ..\gh-pages gh-pages # checkout the gh-pages branch into the local gh-pages folder

```

### Well...

Now, changing from on branch to the other is simple as changing the current directory using ```cd```.

```powershell
λ  dir


    Directory: C:\Users\sacha\Sources\<my-root-folder>


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       22.12.2017     10:40                gh-pages
d-----       22.12.2017     10:39                work


C:\Users\sacha\Sources\<my-root-folder>
λ  cd .\gh-pages\
C:\Users\sacha\Sources\<my-root-folder>\gh-pages [gh-pages ≡]
λ  git status
On branch gh-pages
Your branch is up-to-date with 'origin/gh-pages'.

nothing to commit, working tree clean
λ  cd ..\work\
C:\Users\sacha\Sources\<my-root-folder>\work [master ≡]
λ  git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working tree clean
```

