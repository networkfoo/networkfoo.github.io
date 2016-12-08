---
layout: post
title:  "Fedora meet Jekyll (now play nicely)"
date:   2016-12-07 19:30:04 +0100
categories: jekyll update
---
Where better to start than to write about how to get this whole show on the road.

A nice way to use GitHub Pages is to run Jekyll to handle the layout for your posts. 

But first, you will have to setup your GitHub Pages site, so go [here](https://pages.github.com) and return back once that is done.

Follows is a brief installation process under Fedora:

    dnf install ruby-devel rubygem-bundler rpm-build libffi-devel
    gem install jekyll

Once that installs you can change out from root and set up a jekyll directory in your home directory to build your site:

    mkdir ~/Documents/jekyll
    cd ~/Documents/jekyll

Next we create and start a new site:

    /usr/local/bin/jekyll new networkfoo.github.io
    cd networkfoo.github.io
    /usr/local/bin/jekyll serve &

Navigate to [localhost:4000](http://localhost:4000)  and from here you should be able to glean enough information to edit the example site to your own wishes.

The last bit to this puzzle (albeit, one that offers only a modicum of challenge), is to load this up to GitHub. Just dump the contents of the working directory `networkfoo.github.io` into your repository up on GitHub and then navigate to [networkfoo.github.io/index.html](http://networkfoo.github.io/index.html).
