---
title: Displaying version of Ruby & Rails for your Rails application. 
layout: post
published: true
category: programming
tags: [ruby, rails]
comments: true
---

There was a requirement in the Rails project I was working on, to display application's technical information like Ruby version & Rails version which the application is using in an "Application Info" page in the production servers which was having many RoR applications running on different Rail's version.

So, to do the above task while searching a solution for it, I came across few constants provided by Ruby Language & Rails Framework with the help of which the above task can be done easily and it will be valid even if we upgrade the version of Ruby or Rails.

**1) Displaying Ruby Version :-**

Module in Ruby provides several useful constants with the help of which detailed Ruby version can be displayed.

To see all the constants provided by Module enter the below command in irb or rails console :

	Module.constants

To see the list of constants helpful in determining the ruby version of an application, enter the below command :

	Module.constants.collect{|c| c if c.to_s.include?('RUBY')}.compact

<p align="middle">
    <img src="/assets/images/ruby_rails_version/module_constants.png" alt="Module Constants" class="img-responsive img-thumbnail">
</p>

All these constants are pretty much self explanatory :

<p align="left">
    <img src="/assets/images/ruby_rails_version/constants_value.png" alt="Constants Value" class="img-responsive img-thumbnail">
</p>

**2) Displaying Rails version :-**

Module Rails itself provide constants with the help of which Rails version for an application can be determined easily.

To see all the constants provided by Rails Module enter the below command in rails console :

	Rails.constants

Out of all the constants VERSION can be used to do the task. VERSION is a sub-module inside Rails module. To list the constants of VERSION module, enter the below command :

	Rails::VERSION.constants

<p align="left">
    <img src="/assets/images/ruby_rails_version/rails_version_constants.png" alt="Constants Value" class="img-responsive img-thumbnail">
</p>

These constants as well are pretty much self explanatory :

<p align="left">
    <img src="/assets/images/ruby_rails_version/rails_version_constants_value.png" alt="Constants Value" class="img-responsive img-thumbnail">
</p>

You can use these constants directly in your application to display the required information .
