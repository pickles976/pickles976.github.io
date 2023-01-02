---
layout: post
title: Writing an Inverse Kinematics Solver for Fun
---

![](/images/ik_post/pic1.png)

I recently decided to read the Neural Networks From Scratch in Python book, also known as [NNFS](https://nnfs.io/). I struggled a bit with the backpropagation chapter. Either because some of the wording was ambiguous (when referring to "the next" layer, is this the next layer in forward pass or backward pass?) or because the concept is just hard to grasp at first. I don't blame the authors, the book is an amazing crash course in deep learning-- I think some concepts are just genuinely hard and need to be digested for a while. So I decided to tackle backprop from a different angle and watch Andrej Karpathy's lecture on [backprop and gradient descent](https://www.youtube.com/watch?v=VMj-3S1tku0). Thinking in terms of a graph instead of a gradient helped me understand the math, and when I dove back into NNFS with this more intuitive understanding, everything clicked and I was off to the races.

## Robotics

I also started reading [Modern Robotics](http://hades.mech.northwestern.edu/images/7/7f/MR.pdf) and started learning about IK. When I got to the part about homogeneous transform matrices and the jacobian matrix, I immediately recognized that it was a gradient descent problem! 

A homogeneous transform matrix (also called SE(3)) is a 4x4 matrix that contains rotation and position information. If you multiply A x B, the resultant matrix will be A translated and rotated by B.

![](/images/ik_post/matrix.webp)