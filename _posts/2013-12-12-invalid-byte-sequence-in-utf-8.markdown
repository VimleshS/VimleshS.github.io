---
title: invalid byte sequence in UTF-8 
layout: post
published: true
category: programming
tags: [ruby]
---

Often we come across this **#FiveWordTechHorror** **"invalid byte sequence in UTF-8"** while working on Ruby projects or projects built on Ruby Framework (Ex : Rails).

This error occurs when we try to decode any string which has foreign characters (not plain English) which are submitted either through Javascript or say form using Rails and are not properly encoded .

So, the question is how to properly encode & decode ?

1. When submitted through Javascript :-

	Use encodeURI or `encodeURI()` or `encodeURIComponent()` method to encode.

	Ex : Suppose name contains unicode characters.

		encoded_name = encodeURI(name)

	On ruby side decode it using `CGI::unescape` method.

	Ex: Suppose encoded_name in params is received as params[:encoded_name]

		name = CGI::unescape(param[:encoded_name])

2. When submitted through Rails form :-

	i) Use `CGI::escape` method to encode it and `CGI::unescape` method to decode it.

	Example:

    <script src="https://gist.github.com/Amit-Thawait/3eae58dfd9816db63f7d749460d1ffe8.js"></script>

	OR

	ii) Use `unpack` method of String class and then `pack` it using pack method of Array class.

	Example:

    <script src="https://gist.github.com/Amit-Thawait/190c6ead64ba6899b9bea32921addb45.js"></script>
