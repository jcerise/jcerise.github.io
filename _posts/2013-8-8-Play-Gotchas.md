---
layout: post
title: Play Framework Setup Gotchas
---

I've been recently fiddling around with the [Play Web Framework](http://www.playframework.com/), and while doing some intial setup stuff, I ran across some little gotchas I thought would be worth sharing, to save some poor soul some grief in the future (maybe).
<!--break-->
First up, I'm running Ubuntu 13.04, so everything here assumes the same.
So, I installed Play by downloading everything, un-tarred it into a
directory, added the play script into my path so I could call it from
where ever, and everything was hunky dory. I set up a new PLay project
using Intellij IDEA, and was off to the races...except when I tried to
run the play script from inside my project. I could successfully run it
elsewhere, but running the play script within a project yielded the
following error:

{% highlight bash %}
/home/jeremy/play-2.1.0/play: 60: /home/jeremy/play-2.1.0/play: /home/jeremy/play-2.1.0/framework/build: Permission denied
{% endhighlight %}

Okay, so it turns out, for some reason, I needed to set the file
specified, path-to-play/framework/build, to executable. Weird. Running
this fixed the issue:

{% highlight bash %}
chmod +x /home/jeremy/play-2.1.0/framework/build
{% endhighlight %}

Alright, so that was fixed, now back to actually writing code!

Until...I created some unit tests (and the pre-generated projects
actually come with some as well out of the box), and upon trying to run
them via Inetllij IDEA, I was presented with an odd error stating that
the class ApplicationTest could not be loaded...and all my tests were
failing. Looks like Intellij is not great at running Play tests out of
the box.

The fix I found was the add an additional step before launch (via the
run configuration) to run an external tool, the play script with the test:compile
parameter in this case. This makes Intellij load up play before it tries to run the
tests, which in turn allows the tests run correctly and pass.

So, there you go. The permissions issue might just be something with my
OS, but the Intellij issue is defintely something you might run into
(and it can be found [here](http://youtrack.jetbrains.com/issue/SCL-5152) in youtrack). Hope this helps someone!
