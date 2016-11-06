---
title: Ruby on Rails application with MongoDB on Ubuntu 
layout: post
published: true
category: programming
tags: [ruby, rails, ubuntu, mongodb]
---

The very first question arises is ,What is MongoDB ? and How its different ?

**What is MongoDB :-**

MongoDB is a document-based database as opposed to traditional RDBMS (Relational Database Management System) like Oracle & MySQL.

Mongo DB is a document based database means, it stores data in files (like a text file) in BSON (Binary JSON) format.

In Ubuntu, to install the MongoDB package use the command :-

	sudo apt-get install mongodb

Next, you have to install mongo-ruby-driver.

**Installing Mongo-Ruby-Driver :-**

The MongoDB Ruby driver is the 10gen-supported driver for MongoDB. It's written in pure Ruby, with a recommended C extension for speed.

*10gen* is a software company that has developed and provides commercial support for the open source database MongoDB.

Install the mongo gem (mongo-ruby-driver) by using the below command :-

	gem install mongo

For significantly improved performance, install the bson_ext gem. Using compiled C instead of Ruby, this gem speeds up BSON serialization greatly.

	gem install bson_ext

Let's first check whether the above step are done properly or not.

We will try to make connection object :-

Open a terminal, then open irb

```ruby
require 'mongo'
connection = Mongo::Connection.new
```

<p align="middle">
    <img src="/assets/images/ror_mongodb/mongo_connection.png" alt="Mongo Connection" class="img-responsive img-thumbnail">
</p>

In the context of a Rails application MongoDB provide functionality equivalent to, but distinct from, ActiveRecord. Because Mongo is a document-based database, these mappers are called Object Document Mappers (ODM) as opposed to Object Relational Mappers (ORM).

For Mongo DB, several mappers are available:

* MongoMapper from John Nunemaker
* Mongoid from Durran Jordan
* Mongomatic from Ben Myles
* MongoODM from Carlos Paramio
* MongoModel from Sam Pohlenz
* DriverAPILayer from Alexey Petrushin

All the mappers are build on top of the basic Ruby driver.

I will be describing MongoMapper in this blog.

**MongoMapper :-**

1. Install the mongo_mapper gem using the commnad :-

		gem install mongo_mapper

2. Create a new rails application by skipping Active Record by using the command :-

		rails new mongo_app --skip-active-record

3. Open the Gemfile of your application and ad the following lines to it

		gem 'mongo'
		gem 'mongo_mapper'
		gem 'bson'
		gem 'bson_ext'

4. Now run bundle install from the terminal and try to start the rails console.

	You will get a error stating "Could not find a JavaScript runtime"

	<p align="middle">
	    <img src="/assets/images/ror_mongodb/js_runtime_error.png" alt="JS Runtime Error" class="img-responsive img-thumbnail">
	</p>

	As it is written in error : see [https://github.com/sstephenson/execjs](https://github.com/sstephenson/execjs){:target="_blank"}

	So, you have to include a JavaScript runtime for ExecJS in your application. For this, include the gem therubyracer in your Gemfile (it is not needed for Mac OS X or Windows)

		gem 'therubyracer'

	Now you should be able to start the rails console without anr error (This step is just for checking purpose).

5. Create a new file named mongo.rb in config/initializers folder. This file will be responsible for establishing a connection between your rails application and MongoDB.

	<script src="https://gist.github.com/Amit-Thawait/5742d224d693bd7e38280f9d80b22b85.js"></script>

	'localhost' is the Database host to which your rails application should connect.

	27017 is the default port for MongoDB

6. Now we will try to create a basic scaffold for an entity say User.
    
	`rails g scaffold User name:string age:integer --skip-migration --orm mongo_mapper`

	In the above command, we have skipped migration since we are using Active record and also telling rails to use mongo_mapper as ORM.

Now your basic scaffold application is ready. Start the rails  server of your application and try some CRUD operations.

### Some Configuartion related information for MongoDB :-

The configuration file for mongodb which is mongodb.conf file resides in the location :-
/etc/mongodb.conf

The above file contains the path for data files (Binary JSON files where data is stored) and log file for MongoDB.

You can check the default location of data files in dbpath variable and log files location in logpath variable in mongodb.conf file.

To change the location of the data files and log files ---> just edit the dbpath and logpath variable of mongodb.conf file.

Alternatively, it can also be changed from terminal, by using the commands given below

`mongod dbpath` ------> It gives the current path

`mongod --dbpath <new db path>`
                              
`mongod --logpath <new log path>`


I hope this post will get you started with MongoDB in your rails application.
