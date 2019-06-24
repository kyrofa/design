---
layout: default
title: SROS 2 Policy Generation
permalink: articles/ros2_policy_generation.html
abstract:
  One of the key tenets of ROS 2 is reusability, helping end-users avoid constantly
  needing to reinvent the wheel. This article describes how that is applied in SROS 2,
  such that individual package maintainers can describe their package's communication
  requirements, and end-users can use those descriptions to generate a policy for their
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

DDS enforces access control using a signed permissions document for each domain participant (see the [DDS-Security spec][dds_security], section 9.4). In SROS 2, that permissions document is generated from a ROS-specific policy file that could include the permissions for one or many nodes.

There are a few ways to create that policy file. For example, one could craft it by hand. Another option is to launch the system of nodes for which a policy is desired, exercise it, and run `ros2 security generate_policy` to measure the ROS graph and generate a policy that covers its current state. These are both useful, however, there are issues with both of these methods: they put the onus of security squarely on the shoulders of end users.

To understand this point, a good parallel can be drawn between SROS 2 policies and AppArmor profiles. Consider a web developer putting together a web application. The developer would like to confine the application with AppArmor. However, they quickly learn that not only do they need to confine the little piece they've written in e.g. Python and in which they are an expert, they need to confine the rest of the stack as well: web server, database, etc. Well, they aren't experts in those systems. They don't know what it truly requires, and as time is a premium they don't have time to learn. This typically works out in one of two ways:

1. The developer throwing up their hands and skipping confinement altogether
2. The developer moving ahead with confinement and doing a poor job of it

Thankfully, this is not the status quo in AppArmor. Upstream maintainers understand that they are in a much better position than end users to properly confine the components for which they are responsible. As a result, components of the web stack (e.g. Apache, MySQL, etc.) often ship their own AppArmor profiles or abstractions that can be easily integrated by end users.

SROS 2 is following a similar path, defining a way for upstream packages to specify the communication requirements of the nodes in the package, such that a final policy can be easily generated by the end user.


## Package interface definition

So how do upstream packages specify their communication requirements? They define their interface, which is a high-level description of all the parameters, topics, services, and actions provided or required by each node within the package. This provides all the information necessary to generate a policy (and thus DDS permissions), while also providing the metadata necessary for future developments, such as interface-based design using graphical tools, or static analysis of launch files (e.g. "You're remapping from a topic that isn't being published!").

The package interface is exported from the `package.xml`, and might look something like this:

```xml
<package format="2">
  <name>example-package</name>

  <!-- snip -->

  <export>
    <interface>
      <node name="example_node">
        <parameter name="example_parameter" type="bool" \>
        <topic name="/foo/bar" type="std_msgs/String" publish="true" \>
		<service name="/example_service" type="std_srvs/srv/Empty" client="true" \>
		<action name="/example_action", type="example_interfaces/action/Fibonacci" server="true" \>
      </node>
    </interface>
  </export>
```


## Generating keys and policy for a launch file

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




[dds_security]: https://www.omg.org/spec/DDS-SECURITY/1.1/PDF
