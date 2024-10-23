---
title: Safe And Easy Place to Make Notes
date: 2024-10-23 07:01:00 Z
categories:
- Tools
tags:
- Tools
---

WHEN reinstalling the laptop and working environment,  feel that I have been in a big gap with current learning methods.  Without accurate/detailed info and self summary I have to redo all these search and setup again sooner or later.  Really hate about that.  That's where all these comes from.

1. Jekyll \+  siteleaf  \+ github  for free and easy setup,  beside that the notes/learning records/blogs should be much more safe than hosting on my own devices.

2. VIM/VS Code with jekyll  for local desktop.    In case internet access was blocking or there's no any internet access at all.

# INSTALLATION

In order to install Jekyll,   you need to prepare ruby env first.   On MacOS (Just install Gig Sur, the newest one I can get with my old MacPro 2015),  BrewHome is the necessary precondition, and  how to install all those in a GFW protected area will not be covered in this post.  Just list the items here,   I  wish I can have some time one day in the future to complete all of them.

## Not covered here

* *Free Internet  with secret  VPN*

* *Brew install with local mirrors and proxy*

* *ruby install*

`brew install chruby ruby-install`

I can't fully accurate remember what happened at that time,  but if there's any problem you should add --debug --verbose to check the detail.   If any resource downloading from git\* is blocked ( can not fully control that even with local HomeBrew mirrors as underlying interaction with some kind of API) ,   free land proxy should be appended.

`brew install --debug --verbose chruby ruby-install`

`ALL_PROXY=socks5h://xxx.xxx.xxx.xxx:xxxx brew install --debug --verbose chruby ruby-install`

github related documents  *[here](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)*

siteleaf documents *[here](https://learn.siteleaf.com/getting-started/#gsc.tab=0)*

# DEPLOYMENT

Official docs is always the best source.   Operation sequence suggestion:

1. *[Jekyll local trial first to make it fly.](https://jekyllrb.com/docs/step-by-step/01-setup/)*

2. *[connect to github pages,  ensure push will bring github pages update  eventually.](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)*

3. *[connect github with siteleaf, then you can update your pages with visualization.](https://www.siteleaf.com/blog/connecting-github/)*