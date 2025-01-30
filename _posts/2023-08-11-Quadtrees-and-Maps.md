---
layout: post
title: Fun with Quadtrees
---

![](/images/quadtree/map3.PNG)

I think Quadtrees are cool. Trees are definitely my favorite data structure, hands-down. There is something about Log efficiency that makes them amazing. Tries, Quadtrees, Octrees, are great and simple! I have yet to implement a K-D Tree or a BSP Tree, mostly because I haven't had a reason to.

The main appeal of trees to me, and I hope to everyone else, is their ability to give the illusion of massive size. In a game like Kerbal Space Program you can see the ground in relatively high detail when you are walking on the surface, and also see the entire planet from space. You can land on the other side of the planet and find equally as much detail! How much memory does that planet take up if the whole thing is visible at once then? Most people should already know the answer to this question, it's called LOD. Further objects are rendered in less detail than close ones. But no matter how much more I learn about LOD and tree-based systems for spatial partitioning, the magic never goes away.

Here is an example of a Minecraft mod that adds LOD-based rendering as a skirt around the main area rendered by the game.

[![Minecraft LOD](/images/quadtree/thumb.webp)](https://youtu.be/mRmznYAJli4?t=396 "Quadtrees")

It blows my mind just looking at it. It gives computers this illusion of infinite power, the ability to hold infinite worlds inside of a machine.

Anyways, about a year ago I had a project idea that required me to learn several things at once. I wanted to make a 3D map of the game [Foxhole](https://store.steampowered.com/app/505460/Foxhole/) and host it on the web to show off to people in the official Foxhole Discord and subreddit. I ended up learning Three.js, slippy tiles, and how to write a quadtree!

I had some screenshots on my phone and I remembered them other day. I thought they were pretty so I decided to make a blog post with them.

The [quadtree implementation itself](https://github.com/pickles976/Foxhole-Map-3D/blob/main/src/utils/quadtree.js) (oh God, I just looked at it, this is horrible and unreadable) is dead simple, but it can be pretty challenging to think through the algorithm and figure out what the program should be doing at a given point. Lots of looping and recursive programs have this issue. At the beginning there were lots of issues that I couldn't be sure if they were occuring in my quadtree implementation, my tiling program I wrote in Python, or an issue with my coordinate system in Three.js

![](/images/quadtree/sus.PNG)  
A weird issue with offsets

![](/images/quadtree/susso.PNG)  
A single map region rendering

![](/images/quadtree/woot.PNG)  
Strange issue with only the the bottom row of tiles being rendered, but at varying LODs

![](/images/quadtree/map.PNG)  
Now the whole map is rendering, but also a bunch of "parallel universes"

The random colors are just for contrast, but I think they look really cool. It looks like some sort of board game or kaleidoscope-- and vaguely reminds me of the way land parcels are laid out in a grid in the US.

![](/images/quadtree/waltuh.PNG)  
The developers really did an amazing job with this map. If you look closely you can see that the hexagon in the center is slightly higher than all the rest. I believe this was the first hex they created, and had to fit into the normalized range of the other surrounding tiles, so it was adjusted. 

![](/images/quadtree/shiny.PNG)

I took this image because I thought the specular reflection on the glossy water looked really cool in contrast to the softly-shaded and squishy-looking land. Something about this style of render is really peaceful.

The mesh is generated from a heightmap that someone extracted from the game. The heightmap and texture are just served from a URL, using github as a CDN and organizing the tiles in folders. This is how slippy tiles work for mapbox or Google maps. When a node in the quadtree is active, it uses the xyz position to request a texture and heightmap file from a URL, and uses that to generate a textured mesh at a given offset. Pretty simple.

The actual deployed site uses low-res textures and a lower fidelity mesh and a stolen water shader and sky shader. I wanted it to be able to run on any phone and computer anywhere in the world without much trouble. Unfortunately this has the effect of turning the map into a featureless blob when you zoom out too far.

[Here's a link to the final project](https://www.foxholemap3d.app/)