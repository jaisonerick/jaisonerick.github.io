---
layout: post
title: Setting up Angular JS
---

There are a lot of ways to setup Angular JS and start a new project. Most people I know use
yeoman for that but personally I don't like it very much. I rather use the old school approach, setting
up everything I need for each specific project.

The main reason for that is to use just what I need for each project. I saw a lot of projects
using RequireJS for load 3 to 4 javascript files. It's a massive perfomance loss, appart from
the development effort required for maintain these 4 files.

Here, I'll show a little about my last project setup. I started using RequireJS because it came together
with the package I bought [SmartAdmin] for it, but soon I realize that it would do more harm than good and
took it off. I decide to don't think about minfication and other stuff like that so earlier in the project
anymore. The focus should be all in what I build, not in some technicalities that will make a difference
just when we get live.

Here is my project structure:


    project_root
    +-- index.html
    +-- js
    +-- css
    +-- sass
    +-- _posts

 

[SmartAdmin]: http://wrapbootstrap.com
