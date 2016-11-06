---
title: include vs extend in Ruby 
layout: post
published: true
category: programming
tags: [ruby]
---

**include** : Mixes in specified module methods as **instance methods** in the target class

**extend** : Mixes in specified module methods as **class methods** in the target class

Example:

<script src="https://gist.github.com/Amit-Thawait/3590065312ee8dbac575a09b31ac81a7.js"></script>

<script src="https://gist.github.com/Amit-Thawait/5cf313ada3b441aad6dffe1ae5fc6595.js"></script>

<script src="https://gist.github.com/Amit-Thawait/106cc534acdeaab64fff3821c2a8e245.js"></script>

Now, on executing the below commands, you will get

	ClassThatIncludes.new.module_method 

=> "Module Method: Hi there!"

	ClassThatExtends.module_method

=> "Module Method: Hi there!"

If you include module ReusableModule in class ClassThatIncludes, the methods, constants, classes, submodules, and other declarations gets **referenced**.

**include** is a private method, because it's intended to be called from within the container class/module.

If you extend class ClassThatExtends with module ReusableModule, then the methods and constants gets **copied**. Obviously, if you are not careful, you can waste a lot of **memory** by dynamically duplicating definitions.

**extend** is a public method.

extend - adds the specified module's methods and constants to the target's metaclass (i.e. the singleton class). For ex : 

* if you call Klazz.extend(Mod), now Klazz has Mod's methods (as class methods)

* if you call obj.extend(Mod), now obj has Mod's methods (as instance methods), but no other instance of of obj.class has those methods.