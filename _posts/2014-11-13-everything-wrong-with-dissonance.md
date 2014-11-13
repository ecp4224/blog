---
layout: post
title: Everything wrong with the Dissonance demo
---

Dissonance was a game me and my friends had planned on releasing by the end of this year. (lol ya right :p) It was an action 
RPG with a whole set of game mechanics planned out and a whole story outlined. But we wanted to get our feet wet in the world 
of game design so we settled on making a demo for the game for our 4 week highschool internship project. While the demo was 
overall well recieved at theinternship fair, there were many things wrong with it as well, ranging from the design of the 
level objectives, to the programming, to the presentation.

In this blog post, I'll go over a few things I thought was wrong with the Dissonance demo and what we could of done next time.
I'm doing this as an excerise to look back at mine, and everyone elses mistakes. The Dissonance demo (for the 4 weeks we had)
came out well beyond what I expected, but nothing is ever perfect.

#The Programming
The Dissonance demo ran on top of an engine that allowed for quests and scenes to be scripted. While this made it easy to design
levels and scenes, the API was often times very uninformative and poorly designed.

##Rendering
The rendering used OpenGL 1 to render each sprite on the screen..

..and each 16x16 tile in the map was a sprite..

..I think I don't need to go any futher :I

##The Scene API
Scenes were ran on a seperate thread independant of the rendering/updating thread. Which meant all commands were event based,
and you had to wait for each action to be completed before moving on to the next one.

ex:

```java
        ToZesilia.jeremiah.setWaypoint(new Position(jeremiahTarget.getX() - 2500, jeremiahTarget.getY()), WaypointType.SIMPLE);
        ToZesilia.jeremiah.waitForWaypointReached();
        //More scene code
```

This told the engine to set a waypoint for jeremiah and have him start walking, and wait for him to reach that waypoint. 
While this was useful and made things somewhat easy to read, when the API decided to do things on it's own, then the events
would become useless.

For example, lets say there is an obsticale in the way of jeremiah, so he can't reach the waypoint. That means the scene
will never continue.

Or, lets say the API decides to set the waypoint in a few game ticks, well when you go to call `waitForWaypointReached();`,
it won't have any waypoints **yet**, so it will simply do nothing and return. 

Both of these cases happened alot when scripting and made the benifits and the simplictics of the Scene API useless.

##The World API
The World API was absolutly awful. It's API made things confusing for the scripters and my life a living hell trying to debug.
Because a World can be loaded outside the render thread (which 99% of the time it was), but requires the render thread to load,
it often had to schedule a render tick for loading. Which means again, the world load method was event based, and the 
completion of the method doesn't mean it was loaded. As such, a method was added for the scripters to tell them when the world
was loaded:

```java
    myworld.waitForWorldLoaded();
```

This method would halt until the world was fully loaded and do nothing if the world is already loaded.

This is great! So what was the problem?

Again, it was when the API decided to do it's own thing.

For example, the API had it's own way of fading out the current world and fading in the new world. However, this is often **not**
what the scripters wanted. They would fade out, load and display the new world, wait for it to load, **request** a fade in and
wait x amount of ms for it to compelte. However, the problem was the API already faded in the world, so the request would be 
ignored and the game would sit there for x amount of seconds doing nothing.

Since the scripters did not understand what the API was doing when it loaded the world, they would often assume they did something
wrong and spend time trying to debug something they didn't break. 

##World Caching
...

World Caching was an attempt to minimize the memory usage by only keeping a certain number of worlds loaded at any given time.
However, the way it would restore a world from the cache would only create problems and cause duplicate enemies to appear,
the world being empty because it was already unloaded but still in the cache, ect..

Long story short, the World Caching system was very buggy and poorly designed, which lead to major bugs when it came to world
switching. More time should of been spent programming and planning the caching system to avoid these types of problems.

So in conclusion, most of the API's faults were where the API failed to communicate what it was doing, assumed when it 
shouldn't haved, and it's poor programming/planning.

#The Presentation
The intro to the game came out very nice, but what we didn't account for was the length of it. At the fair, students would only
really sit down to **play** our game, they did not care about the backstory of the world or the environment simply because
they didn't have time to. We designed the intro scene as if a player at home would sit down and play through the demo, not
play it at a fair.

However, we couldn't just throw the player in and tell them to go at it, we need to give them a reason to.

The solution? We simply add a skip option for the intro scene that would skip to the office scene. The office scene provides
enough dialog for the player to show who the bad guy is, who the good guy is, and what their objective is.

This simple option would of made a lot of players happy during the first few minutes as they did not really care about
the backstory nor did they care about reading the slow scrolling dialog.

##Level objectives
While the level designs were very well done (thanks to our level designer), the objective in each level would sometimes
be unclear. This is very clear in the first level, where you are on the roof.

It's clear you have to fight and kill the first 2 enemies on the roof, as they are the first thing you see when you start
walking right, however what you do after that is unclear. The locked roof door, while meant to provide a hint as to where
to go, did so poorly. It only confused the player, as there were no other clear ways to get off the roof. While we tried
our best to make the garbage shoot stick out (blue outline, bright color), it's not obvious simply because...why would you
go down a garbage shoot? The player was never taught to jump, nor does it look big enough for the characters to fit down.

Another thing that confused players were when triggers were actived. Triggers are events that are..triggered..when you step in
them. They were meant for things like, preventing a player from going into a room or activating a trap. They were meant to do
something **that the player couldn't control**. However, this trigger was abused because it was easy to script, and it was used
in every level as an "objective trigger".

When you stepped on the trigger in front of the door, you would lose control, a dialog box would appear and the characters
would walk into the door. This often confused players because they didn't initiate the action, and they would often find it on
accident or while running around in the middle of a fight. Forcing the player to interact with the object makes it feel like 
they activated that thing and they accomplished something.

#Conclusion
The list goes on and on, but most of the items are trivial things that we couldn't have done in the 4 weeks we were given.
While the Dissonance demo came out as a nice game prototype made in 4 weeks, it is **anything but a demo**. I really hope
we can make the Dissonace dream a reality sometime in the near future, and hopefully take the lessons we learned here and
apply them to the actual game.

Goodnight internet, it's 3AM and I got to get up in 6 hours...
