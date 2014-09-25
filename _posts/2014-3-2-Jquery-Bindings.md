---
layout: post
title: Event Bindings in jQuery
---

Recently, I've been working on a Ruby on Rails app for use with [OpenSpaces conferences](http://www.mindviewinc.com/Conferences/OpenSpaces.html). What it basically does, is allow conference attendees to suggest and post new topics to a board space that is separated by time and room location. This is all hooked up with websockets to allow for all connected clients to be kept up to date with what everyone is else is posting.

The basic workflow is that a user clicks into a "spacetime" on the board, and a popup form is shown, which allows the user to enter data about their suggested session. While the user is entering this data into the form, the spacetime becomes locked, so no other users can attempt to enter a session into that same space. Once the user has finished entering their data, the spacetime is unlocked, the topic creation is broadcast to all clients, and the new topic appears on the board. You can see a working example of the app so far [here](http://evening-harbor-6170.herokuapp.com/).

Most of the interaction of the app is accomplished through Javascript (CoffeeScript) using jQuery. For example, one of the first things that happens on page load is that we attach a click handler, bindSpaceTimeClick,  to all spacetimes, such that when they are clicked, they broadcast the event to lock themselves, so no other user can add a topic there while it is locked. The code to accomplish this is as follows:

{% highlight javascript %}
    ###
    Binds a click event to a spacetime. Upon clicking a spacetime, a
    bootstrap modal is opened, pointing to the topic creation form.
    ###
    bindSpaceTimeClick = (e) ->
      $this = $(this)
      spaceTimeId = $this.attr('id')
      room = $this.data('room')
      time = $this.data('time')
      modal.on 'shown.bs.modal', () ->
        updateModal spaceTimeId, room, time
        true
      modal.on 'hidden.bs.modal', () ->
        clearModal spaceTimeId, $this
        true
      modal.on 'ajax:success', (e, data, status, xhr) ->
        newTopicSuccess(e, data, status, xhr, spaceTimeId)
        true
      .bind 'ajax:error', (e, xhr, status, error) ->
        newTopicFailure(e, xhr, status, error)
        true
      modal.modal 'show'
      true
{% endhighlight %} 

The bit we are interested in above is the shown.bs.modal event. When the modal is shown, updateModal locks the spacetime as described above. The code for update modal looks like this:

{% highlight javascript %}
    ###
    Once a modal is fully loaded, add some additional data into it, 
    such as the spacetime ID, time and room. This is also when the lock
    is triggered.
    ###
    updateModal = (spaceTimeId, room, time) ->
      dispatcher.trigger 'lock_spacetime', {space_time_id: spaceTimeId}
      $('#new_topic input#topic_space_time_id').val spaceTimeId
      $('.topic-room').html 'Room/Location: ' + room
      $('.topic-datetime').html 'Time: ' + time
      true
{% endhighlight %} 

Looks good, and in theory works well. At this point, more astute readers may notice that something is probably off, and indeed it is. When we first went to test this code, it worked as intended. You can click on a spacetime, and once the Bootstrap modal window appears, the spacetime will be locked (in our app, the table cell it resides in gets grayed out). Closing the modal unlocks the spacetime accordingly. Great, that's exactly what we want. But, if you were to click into a second spacetime, the new one gets locked, but so does the one you previously clicked. This continues to happen until you refresh the page, with each spacetime click locking all previously clicked on cells.

This stumped us for a while. We went down several routes to try and figure out just what was going on. We originally thought our websocket dispatcher was broken, and that our events were being queued and never cleared. We thought maybe our event router was duplicating calls for some reason. None of these turned out to be the problem. After a couple of hours of staring and tweaking and watching things in dev tools, we discovered our error, and what a small error it was. Our click event binding was broken, and was basically re-binding our shown.bs.modal event on each click, causing the events to stack.

Since we had put the modal.on bindings inside the click handler, every time someone clicked into a cell, the modal.on binding was getting re-applied, meaning that we were adding a new event handler with each click. This perfectly explained the behavior we were seeing. Basically, we were trying to do too much in our click handler. We desperately needed a re-factor. So, what we did was move the modal.on event bindings out of the click handler, and shuffled a few things around to make the click handler simpler. Basically:

{% highlight javascript %}
    ###
    Binds a click event to a spacetime. Upon clicking a spacetime, a
    bootstrap modal is opened, pointing to the topic creation form.
    ###
    bindSpaceTimeClick = (e) ->
      $this = $(this)
      curSpaceTime = $this.attr('id')
      modal.modal 'show'
      true
{% endhighlight %} 

Much simpler, and now we're just setting a module variable called curSpaceTime to maintain some idea of where the user is, and that's about it. Conversely, the updateModal method now looks like this:

{% highlight javascript %}
     ###
    Once a modal is fully loaded, add some additional data into it, 
    such as the spacetime ID, time and room. This is also when the lock
    is triggered.
    ###
    updateModal = (spaceTimeId) ->
      $this = $("td#" + spaceTimeId)
      room = $this.data('room')
      time = $this.data('time')
      dispatcher.trigger 'lock_spacetime', {space_time_id: spaceTimeId}
      $('#new_topic input#topic_space_time_id').val spaceTimeId
      $('.topic-room').html 'Room/Location: ' + room
      $('.topic-datetime').html 'Time: ' + time
      true
{% endhighlight %} 

All the modal specific information is now handled here, instead of upon click, which I think makes a little more sense. Testing this new setup also works like it's supposed to as well. 

Looking back, those more experienced in these things would probably laugh at my noobishness in this scenario, but I think of it as a pretty interesting debug session, with a not, at first, obvious outcome. To conclude, don't bind events within other events unless you want a continually growing stack of events bindings.

If anyone is interested, the app referenced in this post can be found [here](https://github.com/BruceEckel/OpenSpacesBoard)
