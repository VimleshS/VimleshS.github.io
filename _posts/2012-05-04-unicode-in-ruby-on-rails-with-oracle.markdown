---
title: Unicode in Ruby on Rails with Oracle 
layout: post
published: true
category: programming
tags: [ruby, rails, unicode, oracle]
comments: true
---

Few days back, I faced an issue related to unicode characters in my application (Ruby version - 1.8.7, Rails Version - 3.2.2). So, I am highlighting few area's which one should take care of while dealing with unicode characters.

Here's a step by step process which one must ensure ,so that all the characters are displayed correctly.

**STEP 1 :-**

The default encoding of the database should be set to UTF-8.

For Oracle, it can checked  by the below query

SELECT VALUE FROM nls_database_parameters WHERE parameter = 'NLS_NCHAR_CHARACTERSET';

and the result should be

AL16UTF16  or  AL32UTF8

However AL32UTF8 supports latest unicode characters.

For MySQL Database, please refer [encoding in MySQL](http://www.daveperrett.com/articles/2008/06/23/force-mysql-encoding-to-utf8/){:target="_blank"}

**STEP 2 :-**

Encoding in database.yml should be set to utf-8

Add the below line in your database.yml against every environment.

encoding: utf8

**STEP 3:-**

Set 'NLS_LANG' variable in config of your rails application.

ENV['NLS_LANG'] = (NLS_LANGUAGE)_(NLS_TERRITORY).encoding

The value to which the above variable(ENV['NLS_LANG']) has to be set can be determine by the result of the below query

select * from nls_database_parameters;

From the result of the above query, concatenate the value of  NLS_LANGUAGE and NLS_TERRITORY with a underscore followed by encoding. Two examples are given below.

(NLS_LANGUAGE)_(NLS_TERRITORY).UTF8

or 

(NLS_LANGUAGE)_(NLS_TERRITORY).AL32UTF8

for my case, the value of NLS_LANGUAGE and NLS_TERRITORY were 'AMERICAN' and 'AMERICA' respectively.That's why, I set

 ENV['NLS_LANG'] = 'AMERICAN_AMERICA.UTF8'

Note : AL32UTF8 supports latest unicode characters.

You can set this 'NLS_LANG' variable either in config/application.rb or config/boot.rb or config.environment.rb or config/preinitializer.rb
(However by default, preinitializer.rb file does not get created while creating a rails app, you have to create it manually.)

**STEP 4 :-** (If you want to write in a PDF file)

If you want write in a PDF file then add the meta tag at the top of your view file as

<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

However the same is not required while writing to a excel file.

But there might be some characters which might not get displayed properly (for ex : ₹ (Indian currency Rupee symbol)) in the excel sheet, so better add the above meta tag at the top of your view file related to excel as well.

**STEP 5 :-** (Internet Explorer specific issue)

Say, if  your user's name contain foreign characters and you want to generate a PDF report for them and want to save the report with their name then, there will be an issue while doing the same in Interner Explorer but not in Mozilla Firefox.

The same can be resolved using the Iconv class of Ruby. You can use the below method for the same

<script src="https://gist.github.com/Amit-Thawait/171770f8a7c2f589de9aacbd2a849081.js"></script>

and add the below line at the top of your class declaration

```ruby
require 'iconv'
```

For Ex:

<script src="https://gist.github.com/Amit-Thawait/fc05a90fd8d26fc06128e97cba38f131.js"></script>

Iconv is a wrapper class in Ruby, which translates string between various encoding systems.

The above method will convert UTF-8 strings to Windows-1252 character set.

Some people also use ISO-8859-1 character set instead of Windows-1252, however Windows-1252 is better as it contains more characters than ISO-8859-1.

**Sorting of objects containing unicode characters :-**

Array of objects with unicode characters cannot be sort with array's sort method directly, for ex :
Say, you have @users as array of user objects then you can use the above method to get the sorted array of user objects based on user's name, then by using normal sort method we can write

```ruby
@user = User.all.sort{|a,b| (a.name <=> b.name)} --> Ascending order
@user = User.all.sort{|a,b| (b.name <=> a.name)} --> Descending order
```

But the above line will fail, if  user's name contains special character because array's sort method sorts basing on ASCII value and special characters have higher ASCII value. So all the names with special character will appear at the bottom after sorting for every respective alphabet.

However there are 2 ways to solve it :-

**1) Using Rails Internationalization (I18n) API :-**

This API converts the given string into the specified locale. By default the locale is set to English. You can change it as well.

For ex :

```ruby
I18n.transliterate("Jürgen") # => "Jurgen"
```

So, only thing you have to do is to use the transliterate method of I18n API.

```ruby
@user = User.all.sort{|a,b| (I18n.transliterate(a.name) <=> I18n.transliterate(b.name))}
```

That's it.

For more details on transliterate method of Rails Internationalization (I18n) API you can check the link [Transliterate method of ActiveSupport::Inflector Module](http://api.rubyonrails.org/classes/ActiveSupport/Inflector.html#method-i-transliterate){:target="_blank"} and for more details of I18n API refer [Rails Internationalization (I18n) API](http://guides.rubyonrails.org/i18n.html){:target="_blank"}

**2) Using nlssort function of Oracle :-**
There is a function in oracle which can be used for the same which is

```sql
nlssort(name, 'NLS_SORT=GENERIC_M')
```

Say, you have @users as array of user objects then you can use the above method to get the sorted array of user objects based on user's name. For example

```ruby
@users = User.find(:all, :order => "lower(nlssort(name, 'NLS_SORT=GENERIC_M'))")
```

For more details about the nlssort function of oracle you can check [The NLSSORT Function](http://docs.oracle.com/cd/B28359_01/server.111/b28298/ch9sql.htm#i1006311){:target="_blank"}


I hope the above steps will help you, if you are stuck with unicode issue with Oracle for Ruby on Rails.

