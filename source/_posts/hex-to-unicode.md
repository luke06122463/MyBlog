title: Hex to unicode
date: 2015-08-08 00:27:05
category: RubyOnRails
tags:
---

Sometimes, we need to read data from old-fashioned devise. Unfortunately, many old-fashioned devise doesn't support unicode very well. So we need to translate data from any format to unicode.

<!--more-->

Now things become clear that the string returned by ldap API is hex rather than Unicode. We need to format these string to utf-8 before taking any advantage of them.

There are mainly two ways to use ldap result:

1. show the ldap result on web client
2. store the ldap result to elasticsearch

I write a common function to finish the formatting work and here is its definition:
![Alt text](/img/unicode_001.jpg)

You can use the function to format every ldap result with the usage of *Format::String.format_json_to_unicode(source)*
