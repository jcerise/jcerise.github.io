---
layout: post
title: Generating realistic (two dimensional) caverns with cellular automata
---

I have been building a roguelike game in python for a while now. For
those not familiar with this particular type of game, I will point you
[here](http://en.wikipedia.org/wiki/Roguelike).

Basically, its, as my
girlfriend put it, "About the nerdiest thing you can possibly do with a
computer". Opinions aside, roguelikes are very enjoyable to write, as
I've found, as you can focus solely on gameplay, and not worry about
fancy rendering engines and lots of 3D math.
<!--break-->
One of the staples of a roguelike is the dungeon generation algorithm.
Since 99% of a traditional roguelike is spent crawling through a dungeon
in search of monsters to slay and epic loot to pocket, its rather
important that the player is presented with an interesting series of
paths and environments to traipse through. Many roguelikes challenge the
player with navigating exclusively dungeon environments (straight
corridors, connecting rooms, which in turn are usually rectangular in
shape), and not much else. To be sure, my roguelike ([DungeonCrawler](https://github.com/jcerise/DungeonCrawler)),
employs this formula, but I felt it lacked some variety. It gets tedious
constantly exploring the same environment over and over again. So, after
a little thought about what other types of subterranean labyrinth I
could throw at the player, I settled on caverns. Caverns present an
interesting challenge, as they need to look natural, which means no
straight lines, and also unpredictable corridors and rooms.

I ended up implementing a celluar automata algorithm to accomplish my
cavern building. Cellular automata, if you are unfamiliar, is the
concept behind Conways Game of Life. Basically, each tile in our cavern
can have one of two states: floor or wall. We start out by randomly
filling our entire map with tiles. I use the ratio of roughly 40% wall
tiles to 60% floor tiles (this tends to generate the nicest looking
results, in my experience).

{% highlight python %}
for x in range(0, len(self.map)):
    for y in range(0, len(self.map[x])):
        if randrange(0, 100) < 42:
            self.map[x][y].blocked = True
            self.map[x][y].block_sight = True
{% endhighlight %}

<img src="http://i.imgur.com/0IxpKNu.png">

As you can see, it looks pretty chaotic at this point. This will form
the basis for our cavern. For the record, brown tiles are floor, and
grey tiles are wall. The next step is to actually run our cellular
automata algorithm. I started out using the 4-5 rule (if a tile was a
wall and 4 or more of its neighbors are walls, it becomes a wall, or if
it was not a wall and 5 or more of its neighbors are walls). We pass
over each tile, and check its neighbors, and decide if it should be a
wall or a floor, based on the above stated criteria. The code to
accomplish this looks like this:

{% highlight python %}
for _ in range(5):
    for x in range(0, len(self.map)):
        for y in range(0, len(self.map[x])):
            wall_count_one_away = self.count_walls_n_steps_away(self.map, 1, x, y)
            wall_count_two_away = self.count_walls_n_steps_away(self.map, 2, x, y)
            tile = self.map[x][y]
            if wall_count_one_away >= 5 or wall_count_two_away <= 2:
                #This tile becomes a wall
                tile.blocked = True
                tile.block_sight = True
            else:
                tile.blocked = False
                tile.block_sight = False
{% endhighlight %}

We make several passes like this, in my case 5. Each pass will smooth
out the cavern a little bit more, as the code decides, based on the
tiles neighbors, what each tile should be. If there are a lot of walls
around it, it should probably be a wall, and vice versa. Below you can
see the results of this after one pass (comapre it to our starting map
above):

<img src="https://i.imgur.com/xSlEwfh.png">

And the results after 2 passes (these are obviously not the same map, I
had some trouble doing a step by step on one map, but it still
illustrates the idea all the same):

<img src="https://i.imgur.com/noUAhsr.png">

As you can see, the cavern starts to take shape, as each tile is changed
to reflect the environment around it. Finally, after five passes, we end
up with this:

<img src="https://i.imgur.com/JzBbeGQ.png">

After our five initial passes, I make 4 more using a slightly modified
algorithm. These passes mainly smooth out the caves a little more, and
get rid of any isolated walls or floor tiles that are left sitting in
the middle of nowhere:

{% highlight python %}
for _ in range(4):
    for x in range(0, len(self.map)):
        for y in range(0, len(self.map[x])):
            wall_count_one_away = self.count_walls_n_steps_away(self.map, 1, x, y)
            tile = self.map[x][y]
            if wall_count_one_away >= 5:
                #This tile becomes a wall
                tile.blocked = True
                tile.block_sight = True
            else:
                tile.blocked = False
                tile.block_sight = False
{% endhighlight %}

After these last few iterations, our cave now looks like this:

<img src="https://i.imgur.com/ESZxsK4.png">

At this point, we have a pretty nice, realistic looking cavern,
but you may notice that there is the potential for rooms that the player
cannot reach, and that are not connected to the main cavern. This is not
ideal, as the player is randomly placed in the cavern when the game
starts, and we do not want him to start in a small cavern not connected
to the main cavern.

To fix this problem, we need to identify each 'cavern' that is part of
the larger cave complex. I define a 'cavern' as a space that is not
connected to any other spaces. So, in the above example, there would be
five caverns, one main, large cavern, and then four smaller caverns
spread out around the edges. Once we have identified the individual caverns in our cave
system, we can then either connect them all (not ideal, as it looks
un-natural), or fill in all but the largest cavern (this is the approach
I take, as it maintains a realistic feel). First, how do we go about
identifying the various caverns that compose our cave system?

I used a flood fill algorithm to accomplish this. But, before we do
anything else, we need to seal up the edges of the map, so the player
cannot wander off the screen (my game does have scrolling maps yet, what
you see is the entire map). This is done as follows:

{% highlight python %}
#Before we do anything else, we need to seal up the edges of the map, so the player cannot wander out into
#nothingness. We do this by walking around the edges of the map and making them all wall
for x in range(self.width):
    for y in range(self.height):
        if x == 0 or y == 0 or x == self.width - 1 or y == self.height - 1:
            self.map[x][y].block_sight = True
            self.map[x][y].blocked = True
{% endhighlight %}

Okay, now that we've done that, lets get on to the floodfill algorithm.
The basic logic behind it is this: We loop through every tile on the map
checking for two things. First, have visited this tile already, and
second, is this tile a wall. If either of those is true, we ignore the
tile and move on. When we find a valid, unvisited, floor tile, we add
the tile to an array that will contain all tiles in the particular
cavern (we know we are in a cavern by the fact that this tile is floor).
Next, using our newly added cavern tile, we pop it off of the array, and
check all of its neighbors. If they are unvisited, and not a wall, they
are added to the cavern array. We run this sequence for each valid
neighbor, and each of their valid neighbors. When the array is finally
empty, we have exhausted all tiles in the current cavern, and now know
every tile that the cavern is composed of (when I pop the tile off the
cavern array, I push it onto a record keeping array, so we have record
of every tile in the cavern). At this point, all tiles in the current
cavern have been visited, so they will be ignored going forward. We
continue in this manner until all individual caverns in the cave system
have been mapped out. The code to accomplish this is as follows:

{% highlight python %}
#Now, begin looping through the map, looking for individual caverns
for x in range(self.width):
    for y in range(self.height):
        #Grab the tile at the current coordinates
        tile = self.map[x][y]

        #Set up some empty arrays to hold our current cavern
        cavern = []
        total_cavern_area = []

        #Ensure this is a non-wall tile that has not already been visited
        if not tile.visited and not tile.is_wall():
            #If it meets the criteria, add it to the new cavern
            cavern.append(tile)

            #Loop through all potentially valid cavern tiles for this cavern, and see if they are actually part
            #of the cavern or not. If they are, add them to the total, and grab all four of their neighbors
            while len(cavern) > 0:
                #Get the last item in the candidate list
                node = cavern.pop()
                if not node.visited and not node.is_wall():
                    #Mark the tile as visited
                    node.visit(True)
                    total_cavern_area.append(node)

                    #Append the tile to the west to the cavern array
                    if node.x - 1 > 0 and not self.map[node.x - 1][node.y].is_wall():
                        cavern.append(self.map[node.x - 1][node.y])
                    #Append the tile to the east to the cavern array
                    if node.x + 1 < len(self.map) and not self.map[node.x + 1][node.y].is_wall():
                        cavern.append(self.map[node.x + 1][node.y])
                    #Append the tile to the north to the cavern array
                    if node.y - 1 > 0 and not self.map[node.x][node.y - 1].is_wall():
                        cavern.append(self.map[node.x][node.y - 1])
                    #Append the tile to the south to the cavern array
                    if node.y + 1 < len(self.map[x]) and not self.map[node.x][node.y + 1].is_wall():
                        cavern.append(self.map[node.x][node.y + 1])

            #Cavern detection and construction completed, so append this cavern to the list of all caverns
            caverns.append(total_cavern_area)
        else:
            #This was not a valid cavern candidate, so mark it as visited so we dont bother with it again
            tile.visit(True)
{% endhighlight %}

The caverns variable is a list off all detected individual caverns. So,
the end result is that we now know about every individual cavern on the
map, and can act on them accordingly. The next thing we do is to sort
the caverns by smallest to largest. This way, the largest cavern
(usually what we want to be the main cavern) will be last, and we can
remove it from the list of caverns. Since we are going to fill in all
but the main cavern, we want to ignore it, so removing it from the list
is the easiest and safest way to do this. Then, we simply fill in all
the remaining caverns:

{% highlight python %}
#Sort the cavern arrays so the largest cavern (the main cavern) is the last item, then remove it from the list
#All the remaining caverns will be filled in
sorted_caverns = sorted(caverns, lambda x,y: 1 if len(x)>len(y) else -1 if len(x)<len(y) else 0)
main_cave = sorted_caverns.pop()

#Fill in each of the remaining caverns, as they are not part of the main cave. This will ensure that every
#part of the cavern system is accessible to the player
for cave in sorted_caverns:
    for tile in cave:
        tile.blocked = True
        tile.block_sight = True
{% endhighlight %}

And finally, we are left with one main cavern that player can reach
every part of:

<img src="https://i.imgur.com/UWq6SQi.png">

Pretty nice, if I do say so myself. I've included some more examples of
the final product below. Feel free to comment with any questions or
critiques in the comments below. I hope this has been helpful, or at the
very least midly interesting. YOu can also check out my roguelike in
progress on github [here](https://github.com/jcerise/DungeonCrawler), so
feel free to check out the code and give it a shot. I'm also open to
pull request as well, if you feel so inclined.

<img src="https://i.imgur.com/jQxDlsN.png">

<img src="https://i.imgur.com/lrpak1g.png">

<img src="https://i.imgur.com/hFMnSSk.png">
