---
title: Failed to build gem native extension error while installing mysql & pg gem on ubuntu 
layout: post
published: true
category: programming
tags: [ruby, rails, gem]
---

If you are facing "Failed to build gem native extension" error while installing mysql and pg gem on ubuntu, which might look like

<p align="middle">
    <img src="/assets/images/mysql_pg_gem_install_issue/install_issue.png" alt="Gem Install Issue" class="img-responsive img-thumbnail">
</p>

then you need to install the below mentioned packages for the same :

**For mysql gem :**

	sudo apt-get install libmysqlclient-dev

**For pg gem :**

	sudo apt-get install libpq-dev

After installing the above packages, try installing the gem again. Hopefully, this time, it will get installed successfully.

