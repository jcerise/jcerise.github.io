---
layout: post
title: Learning Godot, Part 1: Extending the Pong Tutorial
---

In my never-ending quest to build a game, I have recently started fiddling
around with the [Godot engine](http://www.godotengine.org/). Godot is, to draw
a parallel (and possibly an unfair one at that), kind of an interesting mash up
between [GameMaker: Studio](https://www.yoyogames.com/gamemaker), and
[Unity](https://unity3d.com/). It can handle both 2D and 3D games, although
it currently really shines for building 2D games. Its supports a relatively
robust, and very Python-like scripting language. And, best of all, its
open source. There are a great many other features that would potentially draw
someone to Godot, but I'll save those for another post.
<!--break-->
Godot, however, suffers from that old open source blight of relatively thin
documentation (although it does seem to be getting better over time). To be
sure, there is a [wiki](http://docs.godotengine.org/en/latest/), which does
have lots of good information, and a full language reference, but I found it
a little lacking in terms of actually getting a full project built using the
engine, start to finish. Which is why this post (and possibly a few to follow)
exists.

From a beginners standpoint, one of the best places to begin on the wiki is the
[Pong tutorial](http://docs.godotengine.org/en/latest/tutorials/step_by_step/simple_2d_game.html).
This short little tutorial goes over the basics of setting up a project, adding
some assets, and moving things around. Once I had completed it, though, I found
the little game I had created to be...lacking. So, I decided to flesh it out into
a full blown game, which included the following features:
* A title screen
* A working menu on said title screen
* A score keeping mechanism in the game
* A victory condition and a win screen
* Some way to exit the game
* A single player mode, alongside the demonstrated two player mode

So, that's what we're gonna do. I assume, at this point, that you have completed
the Pong tutorial linked above, because its going to be the basis for the
remainder of this series. So without further ado, lets jump right in.

The first thing we should add to make this pong game feel more like an actual
game, is some way of keeping track of how each player is doing. In the
traditional pong game, each side has a score tracker that shows the current
points for the respective player. I decided that this was the way to go. First,
here's what we're going to end up with:

<img src="http://i.imgur.com/a7W0bD0.png">

Pretty simple, but it adds quite a bit to our game.

To implement this, I'm taking advantage of Godot's AnimatedSprite node. The
AnimatedSprite is pretty much just what it sounds like, a sprite that has an
animation timeline attached to it. Godot has a wonderful little animation
editor, but that's a little beyond the scope of this post. We're just going to
use the AnimatedSprite to store multiple frames (in this case, each number for
the score), and show them as the players scores increase. The idea is that each
time some one scores, we can set the frame of the AnimatedSprite to the correct
value.

To do this, we first need to add a new AnimatedSprite node to our base node.
I've called mine 'player_score':

<img src="http://i.imgur.com/6H1hu1V.png">

Once your player_score node is created, in the node properties for AnimatedSprite,
you'll find a 'Frames' option. In here, we select 'New SpriteFrame':

<img src="http://i.imgur.com/QvW0E4s.png">

And then clicking the little (and frankly easy to miss) arrow next to the dropdown,
we are presented with a screen where we can load up all the different frames for
our sprite. Clicking the folder icon will allow you to load:

<img src="http://i.imgur.com/sFqe3LL.png">

A quick aside here: I use a small program called [Aseprite](http://www.aseprite.org/)
to create all the assets in this tutorial. Its a wonderful little program for
creating pixel art, and I highly recommend it. You can find all the assets I
created for this post [here](http://bit.ly/1oitBom), but I would encourage you
to create your own, as mine are more or less terrible.

At this point, we should now have an AnimatedSprite node that can be dropped into
our scene. You can position it wherever you see fit, and play with the scaling,
but I put mine just to the left of the center line. You'll notice when you place
it in the scene, it just shows the first frame:

<img src="http://i.imgur.com/PNpwviV.png">

Well, that's pretty cool, but how do we actually get it keep track of the current
score? We'll need to do a bit of scripting for that. First things first, we'll need
to define a couple of variables to keep track of both the player and the opponents
scores. We're going to open the script created during the Pong tutorial on the
main game node, and add these two lines right under the screen_size and pad_size
variables:

{% highlight python %}
var player_score = 0
var opponent_score = 0
{% endhighlight %}

This will give us a place to keep track of both players scores. The next thing
we need to do is update those scores when either the player (left side) or the
opponent (right side) scores. We can do that by modifying the code to check when
the ball is off the screen, on either the left or right side. From the tutorial,
we have this to check if the ball is off screen:

{% highlight python %}
if (ball_pos.x<0 or ball_pos.x>screen_size.x):
    ball_pos=screen_size*0.5 #ball goes to screen center
    ball_speed=80
    direction=Vector2(-1,0)
{% endhighlight %}

We're going to change that up a bit, so we know who scored. If the ball went out
on the left side, the opponent scored, and if the ball went out on the right side,
the player scored. I chose to break it up (for now) into two conditionals:

{% highlight python %}
if (ball_pos.x < 0):
		# The opponent has scored, increase his score by one
		opponent_score += 1

if (ball_pos.x > screen_size.x):
		# The player has scored, increase his score by one
		player_score += 1
{% endhighlight %}

Easy. Now, when the ball moves off the screen on either side, the appropriate score
is updated. Just storing that value doesn't really do us much good though. We need
to actually show it the player! This is where our AnimatedSprite comes in handy.
Each frame of the AnimatedSprite can be accessed using a numeric index. frames[0]
is the zero sprite, frames[1] is the one sprite, etc. Hopefully, you see where this
is going. We can simply use the current score as the index for the frames of the
AnimatedSprite, and the AnimatedSprite will show the correct score to the user!
Pretty slick, I think. We'll modify the above code to accomplish this:

{% highlight python %}
if (ball_pos.x < 0):
		# The opponent has scored, increase his score by one
		opponent_score += 1
		get_node("opponent_score").set_frame(opponent_score)
		ball_pos = reset_match(ball_pos)

if (ball_pos.x > screen_size.x):
		# The player has scored, increase his score by one
		player_score += 1
		get_node("player_score").set_frame(player_score)
		ball_pos = reset_match(ball_pos)
{% endhighlight %}

We're using the get_node function, which returns the node object we specify, and
then calling set_frame on it. set_frame does exactly what it sounds like, it sets
the current frame of the AnimatedSprite to the index we supply, in this case the
current score. I was a little sneaky here and already added the opponent_score
node, but it is essentially identical to the player_score node. You'll also
notice I added a call to a function, reset_match, which takes the ball_pos, and
returns a new ball_pos. The code for this is:

{% highlight python %}
func reset_match(ball_pos):
	"""
	Resets the game after a player has scored
	"""
	ball_pos = screen_size * 0.5
	ball_speed = 80
	direction = Vector2(-1, 0)

	return ball_pos
{% endhighlight %}

I just took the code to reset the game, and put it in a function, since it is
being reused in several locations.

At this point, you should be able to fire up your game, and watch the scores
increase each time someone scores. Neat!

This is a good stopping point, but stay tuned! Next time, we'll implement a simple
splash screen and menu for our game. Thanks for reading!
