---
title: 'String in Python'
date: 2019-03-28 06:47:23
lastmod: 2020-04-30 09:30:54
draft: false
tags:
  - 'Python'
categories:
  - 'Python'
aliases:
  - '/string-in-python/'
---

In computer science string is a sequence of characters and characters can be Alphabet, Numeric, Alpha-numeric. In computer science string defined by " " or ''. Example
<pre class="wp-block-code"><code>variable = "Hello Python"</code></pre>
Here "Hello Python" is a string value and variable is a variable of Hello Python string.

Everything in Python is an object. The string has few methods. I will discuss every method in this article.

<strong>#Method 1: Capitalize</strong>
<pre class="lang:python decode:true">str.capitalize()</pre>
Return a copy of the string with Capitalize format.
Example:
<pre class="lang:python decode:true ">string_var= "hello python"
string_var.capitalize()
#Output
"Hello python"</pre>
<strong>#Method 2:  Title</strong>
<pre class="lang:python decode:true">str.title()</pre>
Return the string with the title format.
Example:
<pre class="lang:python decode:true ">string_var = "hello python"
string_var.title()
#Out put
"Hello Python"</pre>
#Method 3: istitle
<pre class="lang:python decode:true ">str.istitle()</pre>
Return boolean value. Either True or False. If the string is title format then return True. If not return False.
Example:
<pre class="lang:python decode:true">str_var = "Hello World"
str_var.istitle()
# Output
True

str_var2 = "hello World"
srt_var2.istitle()
#Output
False</pre>
