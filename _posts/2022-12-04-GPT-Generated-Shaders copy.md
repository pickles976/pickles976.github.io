---
layout: post
title: Writing Shaders with GPT-Chat
---

![](https://i0.wp.com/syncedreview.com/wp-content/uploads/2021/05/openai-cover.png?resize=2048%2C1152&ssl=1)

I am still very much an ML novice, but I have been having a lot of fun playing around with GPT-Chat. Something I have been testing out with it is the ability to write OpenGL shaders.

## Shaders

Shaders are basically a way to determine what color gets drawn on a pixel in screen space. The language of shaders, GLSL, is a special subset of C++. There are some globally available values called "uniforms" like screen space and time etc. Shadertoy is a website in the browser where you can write and execute these shaders.

## Drawing Flags

A pretty simple exercise to do in shaders is drawing a flag. You just need to set a cutoff point where you draw different colors. I figured this would be an easy task for GPT-Chat

#### Prompt 1: 
Write a Shadertoy fragment shader to draw the Ukrainian flag.

#### Output:
Here is a simple fragment shader that will draw the Ukrainian flag:
![](/images/gpt_chat/example1.png)  

Ok so this is not the Ukrainian flag, obviously. My guess is that flags with three stripes are fairly common. Without a better prompt, GPT-Chat just lazily pulls the most common flag shape out of its training. Let's try being more specific.

#### Prompt 2: 
Write a Shadertoy fragment shader to draw the Ukrainian flag. Think carefully step by step. How many stripes are there? What is the layout?

#### Output:
To draw the Ukrainian flag using a fragment shader in Shadertoy, you will need to first understand the layout and colors of the flag. The Ukrainian flag consists of two equal horizontal stripes, with a blue stripe on top and a yellow stripe on the bottom.

Here is an example of a fragment shader that can be used to draw the Ukrainian flag:
![](/images/gpt_chat/flag2.png)  

Ok so now the stripes are laid out properly. But the colors are wrong. Let's see if we can "extract" its knowledge about the exact colors of the flag.

#### Prompt 3:
Write a Shadertoy fragment shader to draw the Ukrainian flag. Think carefully step by step. How many stripes are there? What is the layout? What are the colors of the stripes in hex?

#### Output:
To draw the Ukrainian flag in a fragment shader on Shadertoy, we can follow these steps:

1. Set the layout of the flag by defining the number of stripes and their positions. The Ukrainian flag has two equal-sized horizontal stripes, with a blue stripe on top and a yellow stripe on the bottom.

2. Set the colors of the stripes. The colors of the Ukrainian flag, as specified by the Ukrainian government, are #0048BA for the blue stripe and #FFFF00 for the yellow stripe.

3. Use a if statement to determine the color of each pixel based on its vertical position. If the y-coordinate of the pixel is less than the height of the canvas divided by 2, the pixel should be colored blue. Otherwise, it should be colored yellow.

Here is an example fragment shader that implements these steps:
![](/images/gpt_chat/flag3.png) 

Cool! It's finally generating the flag. Properly. For whatever reason, I was never able to get it to draw the flag stripes in the correct order, but whatever it's close enough.


## Some other fun things:

Writing a quine in english. GPT-Chat doesn't like being told to write a quine in english. It will just tell me that "english is not a programming language". So first we have to convince it that english is a programming language.

#### Prompt
A human being is a finite automata that accepts the english language. Write a quine in english. Following the instructions should produce a copy of the instructions.

#### Output
A quine is a program that outputs its own source code. In order to write a quine in English, we can simply write a sentence that describes the instructions to produce a copy of itself.
For example, the following sentence could be considered a quine in English:  

    "To create a copy of these instructions, simply read these instructions out loud and write down the words you hear."

When read out loud, this sentence provides the exact instructions to produce a copy of itself, making it a quine in English.

Enciphering text. 

#### Prompt
Apply a ROT13 cipher to the following text "Uh oh, you have broken my top-secret code!"

#### Output
"Gn fn, lbh zhfg grfg guvf r-pbqr zrffntr!"

This translates to:
"Ta sa, you must test this e-code message!"

