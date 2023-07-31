---
layout: post
title: Full Stack Footguns
---

![](https://what.thedailywtf.com/plugins/nodebb-plugin-emoji/emoji/tdwtf-emoji/footgun-2.png?v=grn1kta83uc)

This blog post won't be very interesting to anyone who has been a dev for more than like, a year, but it's helpful for me to document my own progress. Back in March I put together an app called [ai-arena](https://ai-arena.com/#/multiplayer) the idea is that you can write some code in the browser to control units in a RTS-style game, and then upload that code to a server where the code competes 24/7 online for control of a galaxy.  

I got the idea from my AI class in school where we could upload our code to a competitive ladder for extra credit.  

![](/images/fullstack/space-settlers.png)
A screenshot of "Space Settlers" from CS 4013 at OU

![](/images/fullstack/ai-arena.png)
Screenshot of my game written for HTML canvas  

This project was exclusively a learning exercise for me. I ended up making:

1. [A Game Engine](https://github.com/pickles976/ai-arena) in canvas to run my game, which could be imported as an ES6 module and run both in a webpage rendered to Canvas, or headless in nodejs for running a simulation
2. Code sandboxing inside the game engine to prevent user code from manipulating the game, browser, or node process/environment
3. [A Svelte SPA website](https://github.com/pickles976/ai-arena-app)
4. An In-browser code editor
5. [A Lambda function for user code santization and and testing](https://github.com/pickles976/ai-arena-ladder)
6. [A NodeJS game server for running individual games safely](https://github.com/pickles976/ai-arena-ladder)
7. [A NodeJS Galaxy server for simulating the galactic-scale battles](https://github.com/pickles976/ai-arena-ladder)
8. [A ThreeJS Galaxy for visualization](https://github.com/pickles976/GalaxyThreeJS)
9. A Supabase backend for handling everything database-related  

Looking back on it, there was quite a lot of stuff that I ended up needing to do for this project. But what stands out to me the most are the numerous horrible mistakes I made. So in this blog post I am going to go through all of them, and talk about what I would like to change in the future.

## The Game Server

The game server is designed pretty decently I think. The first piece is the "Campaign Service" that makes calls to the DB and keeps track of the state of the galaxy, sort of like the world map in the game Civilization. When two players have a conflict, the conditions of the conflict are serialized and sent to a Beanstalk queue.  

The next piece is the "Battle Server". This service just polls the queue for conflicts, then reserves them while it executes a simulation of a battle. Whoever destroys the enemy base within the allowed timespan wins. If the simulation lasts too long, then the winner is determind based on some heuristic. If a player's code consistently times out or uses up too much memory, they lose by default. The Battle Server posts the results of each battle back to a different channel on the Beanstalk queue.

The Campaign Service occasionally polls the results channel on the Beanstalk queue for results, then updates the DB with these results. The Beanstalk queue is a great layer between the Campaign and Battle services. The campaign service is heavy on networking requests but low on computational overhead. The battle service is pretty much exclusively computational overhead. Having a queue between them allows for multiple battle services to be spun up and consume from the Beanstalk queue continuously.  

![](/images/fullstack/flowchart.png)
A diagram for those who hate reading

The problems here lie in the implementation. The logic in the Campaign Service is very convoluted. Between checking the queue, pushing to the queue, and updating the database, there is a lot of looping, stopping and starting, and conditions based on the states of the queue. The state of the Campaign Service is spaghetti'ed all over the place and it's not obvious to me what it should look like at any given point. This kind of logic was easy to write, but absolutely horrible to read and amend.  
  
Something else I (very foolishly) didn't anticipate at all were network errors. Timeouts, failures to resolve URLs, I had not written handling for any of those sort of errors. This wasn't an issue when I was deploying to DigitalOcean droplets, but on my RaspberryPi on my apartment wifi, almost every API call should have incorporated some sort of timeout error handling with backoff and retry.  

Lastly, there are a handful of difficult to reproduce errors. Running in Docker makes it even harder to test these errors.  

### How to fix

Implementing a proper service adapter. Currently I just have functions that loosely wrap the Beanstalk and Supabase client libraries. I would add error handling and defined behavior for dealing with connection failures to both of these services. As well as making them async so that they aren't blocking the rest of the application when they hang.

Having some sort of well-defined central state. Instead of having my state be spread out all over the place, I should probably introduce some sort of standardized object to hold everything in one spot. Which leads me to:

Testing! Having a centralized state makes it much easier to write both unit-level tests and regression tests. Writing unit tests for the Battle Service and Campaign Service would help me separate the non-networking logic into its own components. Switching from my hard dependency on the Beanstalkd and Supabase clients to an Adapter layer would let me switch to a dependency-injection model, which would allow me to do mock testing of both services more easily. 

Third, integration tests. This part I am not really sure how I would go about it. Like I said, it's a bit hard to parse through complex logic like this, but maybe reducing the scale of the galaxy to 10-20 stars instead of several thousand, and then walking through the simulation by hand, and hard-coding those states as my test cases.

Finally, logging. Some sort of logging whenever things crash. This would help me diagnose what causes crashes during the long-running games.

Writing all of this down makes me feel pretty embarrassed about just how awful my practices were for this project, but doing things the wrong way helped me really internalize the message of the SOLID and TDD resources that I read later on.

## The Website

The website is very, very ugly. I wanted to challenge myself and write all of the CSS myself. I don't think this was a good idea. Not just because I am bad at CSS, I think challenging yourself to improve is a noble effort, but because I spent most of my time focusing on the CSS and wrangling with it, and let my components suffer for it.

The components should have taken the center stage and been the focus. What are the potential states the components can be in? How do they handle that state. Can they be unit tested for these different states?

There is a lot of ambiguity to the user interaction too. I had a couple of experienced devs try out the website and flat out tell me they had no idea what was going on. This is never a good sign. I should have first made sure that interaction flow made sense and I had states for every possible outcome of a user/networking interaction before I even thought about adding functionality.

Some of the API code is... terrible. A veritable fettucini alfredo of chained async functions. This is partially because of laziness and refusal to write SQL queries, and partially because of laziness and refusal to use await.

![](/images/fullstack/ew.png)
"WHAT THE F*CK IS THAT" -- R. Lee Ermey

### How to fix

Focus on the components first. Having components driven by data would have saved me quite a bite of code reuse for things like popup modals and labels. Unfortunately I was only recommended the great book [Refactoring UI](https://www.refactoringui.com/) recently. [Also this](https://devblogs.microsoft.com/oldnewthing/20230725-00/?p=108482). Make sure the components have unit tests and interaction flow that makes sense.  

Use styling generated by a professional. The [Skeleton UI toolkit](https://www.skeleton.dev/) was recently recommended to me. In the future I would probably build a prototype of a single page of the website with something like this, then worry about adding in routing and networking later.

Focus on networking LAST. Mock the database or API or whatever. Start with UNIT TESTS, then do MOCK TESTS. Not to test features, but just to test that interaction flow.

## Supabase

I don't really have much to say about Supabase. I should have deployed a local version of Supabase for testing and development purposes. I was actually working on this but got bored and moved on from the project after a while. Having a docker-compose that runs the supabase container + all my backend services would make it possible to do integration tests with Github actions I think. To support this I should make that adapter layer I mentioned earlier, which would provide methods for some of the specific interactions the Campaign Service requires, like starting a new "War" and populating the new galaxy with a bunch of stars.  

## The game itself

If I work on this in the future, which I probably will. The game should be made more user-friendly. I don't mean improving the graphics or the actual mechanics of the game (although those could use some work). I mean making the game so obvious to play that an 8 year-old could sit down and get the hang of it, but a 40 year-old could get so ingrossed in the game that they stay up until 2 am playing it.

This is already really difficult to do for most games, and is unrealistic for a coding game, but I have been thinking about what makes things engaging for people.  

1. Immediate feedback. Having a REPL is ideal.
2. Gentle learning curves. Exposing people to one thing at a time, at their own pace, lets them build mastery of the basics quickly and get the "vocab" of the thing down so they can start building more useful abstractions quickly.
3. Recursive complexity. There are games like Factorio or Supreme Commander or even Minecraft that let you abstract away things that you once did manually. Instead of killing cows by hand you have a cow factory. You can pipe that cow factory into your furnace and pipe that into your automatic chest sorter, and now the only piece of the pipeline you need to touch is feeding your cows wheat every now and then. Letting players build complex behaviors as a block and then hiding that abstraction away lets them do a lot of really cool things.  

The REPL piece is the only bit of the game that actually works. You still have to hit "compile" which is pretty annoying, but whatever.

To expand on #2, when coding, it's usually a good idea to see things in isolation. Seeing your ship by itself makes it much easier to understand what's going on. Making a tutorial that introduces you to things one step at a time would be really helpful. Unfortunately the victory conditions and behaviors of the game itself are hardcoded. Separating the logic that checks for game over conditions into the Game Manager class and then providing it with different behaviors would be helpful.  

As for the coding itself, using Javascript with a custom API for the game is a bit clunky and excludes a lot of people. It does have the benefit of giving you "real" practice at coding in JS. But let's be real, there are infinite better options for this than my crappy game. I think a Forth would be a good candidate to replace Javascript for a few of reasons. Now-- hear me out.

Reason 1: I want to learn Forth and implement a Forth interpreter in WebAssembly for the browser. It seems like a nice gentle lang dev project for someone like me who is too lazy to implement a full interpreter for a more complex language.

Reason 2: The recursive/stack nature of Forth makes it a lot easier to implement state machines, which is really all that these little NPCs are. See #3

Reason 3: The basic control flow (or lack thereof) makes it easier for me to build a visual programming system. I think to make things easier for players, it would be nice to provide them with a visual way of programming. Have cards for API calls, cards for basic operations (operations and combinators), and cards for user-defined functions. 

## Conclusion

I was pretty proud of myself when I finished this project after about a month of continuous work, but now looking at it I feel pretty embarrassed. It obviously wasn't a waste of time, I learned quite a bit and most importantly it helped me contextualize a lot of the "programming best practices" stuff that I read afterwards. I can't get the idea of a persistent online galaxy-scale war between code bots out of my head, so I will probably return to this project again at some point in the future, hopefully with this post as a good reference point for me to not make all of the same mistakes again.






