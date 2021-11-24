---
layout: post
title: "How to set up a blog using Jekyll"
date:   2021-11-23 18:00:00 +0800
tags: blog Jekyll
---

This blog documents the steps I took in order to build my first blog on
my Windows laptop. I have never tried to do all the command line and git
on a windows machine before. This should be fun.

<!--more-->

# Setting up essentials

## Install Git on Windows.

Git can be installed and used in Windows 10. You just need to go to
[this website](https://git-scm.com/) and download the latest version of
Git. Follow instructions in the installer, you can get Git all set up on
your Windows machine.

It is worth notice that the latest install of Git include a feature to
add Git bash as a terminal profile in the Windows Terminal.

After the installation, you can verify that Git is running on your
machine by opening up a powershell or cmd, and type git. If you see all
the possible git command, it means you are all set to use Git.

    PS C:\Users\YourUserName> git
    usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
         [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
         [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
         [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
         [--super-prefix=<path>] [--config-env=<name>=<envvar>]
         <command> [<args>]

## Windows Terminal

Windows Terminal is a terminal emulator with a bunch of features
implemented in Windows OS. Many of these features are long waited. They
might be commonly seen in Unix system, but not Windows system. The
features shipped with Windows Terminal include:

1.  Multiple tabs;
2.  Powershell/cmd and Linux bash in the same window.
3.  Highly customizable (key binding, color scheme, background, etc.)

You can simply install Windows Terminal from the Microsoft app store
([link](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701#activetab=pivot:overviewtab)).

## Install Ruby for Jekyll.

Follow this [official
instruction](https://jekyllrb.com/docs/installation/windows/) to install
Jekyll on windows machine without a WSL. Remember to run the
`ridk install` in the last step of the installation.

After the installation of Ruby is finished, you can open Powershell or
cmd and type in the command `gem install jekyll
    bundler` to install Jekyll.

Verify that you have Jekyll functioning by printing Jekyll version
`jekyll -v`.

# Create a new blog using Jekyll

## Why Jekyll?

Jekyll is can produce static site, which is enough for simple blog. It
is simple in that it requires low maintainence (no database, no admin,
no update). The biggest perk to me is that it take markdown as input,
which seamlessly connects to emacs org mode that I am frequently using
to do my own documentation.

## Build a minimal blog using Jekyll

It is very simple to create a new blog, just: `jekyll new PATH`

If you cd into the directory of your new blog, you can see that there
are files created by Jekyll during the initiation.

    Mode                 LastWriteTime         Length Name
    ----                 -------------         ------ ----
    d-----        11/22/2021     14:18                _posts
    -a----        11/22/2021     14:18             61 .gitignore
    -a----        11/22/2021     14:18            444 404.html
    -a----        11/22/2021     14:18            557 about.markdown
    -a----        11/22/2021     14:18           1155 Gemfile
    -a----        11/22/2021     14:18           2035 Gemfile.lock
    -a----        11/22/2021     14:18            181 index.markdown
    -a----        11/22/2021     14:18           2135 _config.yml

Curious to see how the new blog looks like? Just kickstart it with:
`jekyll serve`

And then check it the browser at `127.0.0.1:4000/`.

Note: if you encounter a loadError of `cannot load such file --
    webrick (LoadError)`, run `bundle add webrick` to install webrick.

## Configure your blog

You can configure your blog globally in `_config.yml`.

If you want some more fancy themes to shine up your block, you can check
<https://jekyllthemes.io/jekyll-blog-themes>.

## Add post to your blog

You can simply put markdown under `_posts` directory and it would be
captured by Jekyll automatically. Be aware of the time, you need to
speficy the time zone at moment, so that you can get all your posts out.
In some cases, if the time of the post is later than current time, it
would be skipped by Jekyll.

My current set-up is writing in org mode, and convert to markdown using
ox-pandoc + pandoc.

# Credit

This post is heavily inspired by this nice
[tutorial](https://www.kiltandcode.com/2020/04/30/how-to-create-a-blog-using-jekyll-and-github-pages-on-windows/),
and the theme of this blog is from a public [Jekyll
theme](https://github.com/niklasbuschmann/contrast).
