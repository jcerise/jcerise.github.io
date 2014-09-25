---
layout: post
title: FizzBuzz using Haskell list comprehensions
---

So, I've recently been playing around with Haskell (for the second time, I started in on it a while back, but drifted away for one reason or another), and I've been having a great time of it. I'm just to the point where I'm playing around with list comprehensions.

List Comprehensions, for those not familiar, are basically the same as set comprehensions in mathematics. That is to say, they can be used to build lists out of other lists. A list comprehension is composed of an output, a list (or set of lists) to draw from, and optionally a set of predicates (conditions). A simple list comprehension to triple all numbers in a list might look like this:

{% highlight haskell %}
[ x * 3 | x <- [1..20]]
{% endhighlight %} 

Which would output the new list:

{% highlight haskell %}
[3,6,9,12,15,18,21,24,27,30,33,36,39,42,45,48,51,54,57,60]
{% endhighlight %} 

You can also have more complex outputs if you desire, such as conditional statements:

{% highlight haskell %}
[ if x 'mod' 3 == 0 then "Fizz" else show x | x <- [1..20]]
{% endhighlight %} 

(Please note, for some reason Kramdown won't escape the backticks around my infix functions, so I've replaced them with single quotes, in case you're copying and pasting for some reason)

Which would output the new list:

{% highlight haskell %}
["1","2","Fizz","4","5","Fizz","7","8","Fizz","10","11","Fizz","13","14","Fizz","16","17","Fizz","19","20"]
{% endhighlight %} 

This got me to thinking that I should try to implement the classic [FizzBuzz](http://blog.codinghorror.com/why-cant-programmers-program/) problem using a list comprehension. My attempt is below (Warning, this is probably terrible code...):

{% highlight haskell %}
[ if x 'mod' 3 == 0 && x 'mod' 5== 0 then "FizzBuzz" else if x 'mod' 3 == 0 then "Fizz" else if x 'mod' 5 == 0 then "Buzz" else show x | x <- [1..100]]
{% endhighlight %} 

And the results are:

{% highlight haskell %}
["1","2","Fizz","4","Buzz","Fizz","7","8","Fizz","Buzz","11","Fizz","13","14","FizzBuzz","16","17","Fizz","19","Buzz","Fizz","22","23","Fizz","Buzz","26","Fizz","28","29","FizzBuzz","31","32","Fizz","34","Buzz","Fizz","37","38","Fizz","Buzz","41","Fizz","43","44","FizzBuzz","46","47","Fizz","49","Buzz","Fizz","52","53","Fizz","Buzz","56","Fizz","58","59","FizzBuzz","61","62","Fizz","64","Buzz","Fizz","67","68","Fizz","Buzz","71","Fizz","73","74","FizzBuzz","76","77","Fizz","79","Buzz","Fizz","82","83","Fizz","Buzz","86","Fizz","88","89","FizzBuzz","91","92","Fizz","94","Buzz","Fizz","97","98","Fizz","Buzz"]
{% endhighlight %} 

So, there you go. My attempt at FizzBuzz using a Haskell list comprehension.