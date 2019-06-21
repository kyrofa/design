---
layout: default
title: SROS 2 Profile Generation
permalink: articles/ros2_profile_generation.html
abstract:
  One of the key tenets of ROS 2 is reusability, helping end-users avoid constantly
  needing to reinvent the wheel. This article describes how that is applied in SROS 2,
  such that individual package maintainers can describe their package's communication
  requirements, and end-users can use those descriptions to generate a profile for their
  system without needing to be experts in every component being used.
author: '[Kyle Fazzari](https://github.com/kyrofa)'

published: true
categories: Security
---

{:toc}

# {{ page.title }}

<div class="abstract" markdown="1">
{{ page.abstract }}
</div>

Original Author: {{ page.author }}

## OUTLINE (delete me)

- Background
    - Policies are a good analague to apparmor
    - Consider: you're a web developer, putting together a web application.
    - You decide you want to confine it, using apparmor
    - You don't, however, know apparmor. So you need to learn it.
    - And now you need to generate profiles not only for your own little slice of the web stack but for EVERYthing. Apache, PostgreSQL, everything you're using.
    - Now ask yourself: are you actually doing to do that? No, of course not, time is a premium.
    - This is why individual components oftentimes ship their own apparmor profiles (e.g. Apache). They know they are in the best position to define one, being maintainers, and they know the best way to get folks to lock down their applications is to help them get there by solving a piece of it for them.
    - SROS 2 is following a similar path, defining a way for upstream packages to specify the communication requirements of the nodes in the package, such that a final profile is as easy to generate by the end-user as possible
- The package manifest
    - foo
