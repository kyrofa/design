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
    - And now you need to generate profiles not only for your own little slice of the
	  web stack but for EVERYthing. Apache, PostgreSQL, everything you're using.
    - Now ask yourself: are you actually going to do that? No, of course not, time is a
	  premium.
    - This is why individual components oftentimes ship their own apparmor profiles
	  (e.g. Apache). They know they are in the best position to define one, being
	  maintainers, and they know the best way to get folks to lock down their
	  applications is to help them get there by solving a piece of it for them.
    - SROS 2 is following a similar path, defining a way for upstream packages to
	  specify the communication requirements of the nodes in the package, such that a
	  final profile is as easy to generate by the end-user as possible
- The package manifest (this might deserve it's own design proposal)
    - First, let's discuss how upstream packages can ship the communication details necessary for end-users to generate good profiles.
	- First of all, packages COULD just ship profiles for the nodes contained within.
	- However, rather than use that, packages can provide a manifest that describe the
	  parameters, topics, services, and actions provided or required by each node within
	  the package. This provides all the information necessary to generate DDS profiles,
	  while also providing the metadata necessary for future developments, such as
	  interface-based design using graphical tools and static analysis of launch files
	  (e.g. you're remapping a topic that no node is publishing)
    - An example of what that might look like
	- (Export from package.xml, include in package.xml, or make completely different?)
- Utility for generating launch keys and profile
    - There are a few different pieces of information necessary generate keys and to
	  statically generate a profile:
        - All nodes (and their packages) involved in the system
		    - Provided by launch files
		- Knowledge of which topic/service/action/parameter each node requires, and the
		  type of access required
		    - Provided by manifest
		- Knowledge of what has been remapped where
		    - Provided by launch files
	- Create a keystore, and call
	  `ros2 security secure_launch_file path/to/keystore path/to/file.launch.py` and
	  assuming all nodes used in the launch system are properly covered by a manifest,
	  keys can be generated for all nodes, and a profile can be statically generated
	  that covers the access required by the entire launch file.

## Background

## Package manifest

## Generating launch keys and profile
