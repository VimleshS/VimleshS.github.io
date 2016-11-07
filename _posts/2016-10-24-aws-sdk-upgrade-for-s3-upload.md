---
title: Faster & efficient file upload in S3 using aws-sdk version 2.x 
layout: post
published: true
category: programming
tags: [ruby, rails, aws-sdk, S3]
comments: true
---

In my Rails(4.2.7) application, we were using <a href="https://github.com/aws/aws-sdk-ruby/" target="_blank">aws-sdk</a> ruby gem to interact with AWS S3.

While I was reading about how to upload large video files to S3 efficiently, I stumbled upon this question in stackoverflow : <a href="http://stackoverflow.com/questions/29105178/uploading-large-file-to-s3-with-ruby-fails-with-out-of-memory-error-how-to-read" target="_blank">http://stackoverflow.com/questions/29105178/uploading-large-file-to-s3-with-ruby-fails-with-out-of-memory-error-how-to-read</a> in stackoverflow.

We were using an older version of SDK (1.8.1.1), while the latest version is 2.6.6 as of writing this blog.

AWS-SDK version 2.x introduces a new API named Aws::S3::Resource which

1. **uses multipart APIs for large objects**
2. **uses multiple threads to upload parts in parallel, improving upload speed**

To access S3 in SDK 1.x we used to set AWS config as 

<script src="https://gist.github.com/Amit-Thawait/2397a589f741fe1223cf53f6ed775389.js"></script>

However from version 2.x, region has to be mentioned otherwise it will complain about region as

<span class="text-red">"missing region; use :region option or export region name to ENV['AWS_REGION'] (Aws::Errors::MissingRegionError)"</span>

To fix this just mention region in AWS config initializer as:

<script src="https://gist.github.com/Amit-Thawait/5255f9b0c7291238d8ff78b686981917.js"></script>

Another notable change is : I have mentioned logger config option as well for 2.x otherwise AWS-SDK will not log create, delete object loggers in the rails log file.

Creating a file in S3 :
-----------------------

Using AWS-SDK 1.x we used to upload a file to S3 like this :

<script src="https://gist.github.com/Amit-Thawait/7eab1936be8970d557be1bf7bffc97f9.js"></script>

where `AWS_CONFIG` is Hash which contains environment specific AWS bucket name.

	ENV['RAILS_ENV'] = "development" unless ENV['RAILS_ENV']
	AWS_CONFIG = YAML.load(File.read("#{Rails.root}/config/aws.yml"))[ENV['RAILS_ENV']]

Using AWS-SDK 2.x, using the new Resource API, upload statement looks like this:

<script src="https://gist.github.com/Amit-Thawait/ec8b640c3c9de2cc85b80e9836e5d497.js"></script>

In the newer Resource API, the method to create a file in S3 bucket is put_object and it has various options that you can send along.

Refer <a href="http://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Bucket.html#put_object-instance_method" target="_blank">http://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Bucket.html#put_object-instance_method</a> for detailed list of options.

Deleting a file in S3 :
-----------------------

Using AWS-SDK 1.x, command to delete a file is

<script src="https://gist.github.com/Amit-Thawait/b391ac62966fd58fa3aa24da468e92cf.js"></script>

Using AWS-SDK 2.x, it is

<script src="https://gist.github.com/Amit-Thawait/939f1c13ea6f1c93af53ef52cc93867c.js"></script>
