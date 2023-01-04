---
layout: post
title: Writing an Inverse Kinematics Solver for Fun (WIP)
---

![](/images/ik_post/pic1.png)

## Requirements

Basic linear algebra knowledge.

## Background

I recently started learning about robotics. I reached out to my network for some good book recommendations and looked at class curriculums and settled on two main texts: [Modern Robotics](http://hades.mech.northwestern.edu/images/7/7f/MR.pdf) and [Probabilistic Robotics](https://docs.ufpr.br/~danielsantos/ProbabilisticRobotics.pdf)

Modern Robotics begins with rigidbody dynamics and Inverse Kinematics. Some of the math seems daunting but it's actually not that complicated when you start to think in code. One of the key concepts is the Homogeneous Transform Matrix. A homogeneous transform matrix (also called SE(3)) is a 4x4 matrix that contains rotation and position information. If you multiply A x B, the resultant matrix will be A translated and rotated by B.

![](/images/ik_post/matrix.webp)

The "r"s in GDB control rotation. X0, Y0, and Z0 control translation. Different matrices control the rotation on different axes:

![](/images/ik_post/rotation.jpg)

It looks like a lot, but we wil build up to it. For now let's focus on 2D. A 2D transformation matrix looks like the form:

![](/images/ik_post/matrix2.png)

The 2D rotation matrix looks like this in code:

```javascript   
[
    [Math.cos(θ), -Math.sin(θ)],
    [Math.sin(θ), Math.cos(θ)]
]
```

We can pad it with the [identity matrix](https://en.wikipedia.org/wiki/Identity_matrix) to make it into a transformation matrix:

```javascript   
[
    [[Math.cos(θ), -Math.sin(θ), x],
    [Math.sin(θ), Math.cos(θ), y],
    [0, 0, 1]]
]
```

If we have an arm with length "r" that is rotating with an angle of theta, we can represent the coordinate position of the end of the arm like this:

```javascript
    [[Math.cos(θ), -Math.sin(θ), r * Math.cos(θ)],
    [Math.sin(θ), Math.cos(θ), r * Math.sin(θ)],
    [0, 0, 1]]
```

The values of x and y are derived from polar coordinates. x = r * cos(θ), y = r * sin(θ). Notice that this is also the result of the operation:

```javascript   
[
    let A = math.matrix([
        [Math.cos(θ), -Math.sin(θ), 0],
        [Math.sin(θ), Math.cos(θ), 0],
        [0, 0, 1]
    ])

    let B = math.matrix([
        [1, 0, r],
        [0, 1, 0],
        [0, 0, 1]
    ])

    let C = math.multiply(A, B)
]
```

So this means we can represent any sequence of robot arm components with a sequence of transformation matrices. To limit confusion from now on I will refer to the parts of robot arms that move as "joints" and the rigid parts as "link". In your own body your knees and elbows would be the joints, while your forearms and lower legs would be the links.

So we have a way to start with the angles of every joint and calculate the position of the end of the arm. The next step is to figure out how we can do this in reverse. Pick a position and generate all of the arm angles that will get us there.

## Robot Arms

Now that we understand the matrices, let's look at an actual equation for a very basic robot arm:

![](/images/ik_post/arm.webp)

This matrix is actually just a pair of parallel equations. We can use basic algebra and trig to substitute and find out the solutions. This type of arm will typically have 2 solutions for any given problem, since the elbow can be bent at a negative or positive angle. But what if we want to solve for a more complicated arm with infinite solutions, or 100 joints? We can no longer solve algebraically in that case. We now have to treat this problem like an optimization problem.

There are lots of ways to solve IK as an optimization problem. [This page](https://motion.cs.illinois.edu/RoboticSystems/Optimization.html) has a lot of really good examples. I initially went with gradient descent.

## Gradient Descent

Gradient Descent is not a difficult concept by itself, I will try to explain it with as little math as possible. You will just need to remember the definition of a derivative:

![](/images/ik_post/derivative.png)

Imagine that you are blindfolded on top of a large hill or mountain, and you need to find your way to the bottom of it. One possibility would be to take a step north (+X) and a step east (+Y). If your step north is downhill, then move north. If your step east is downhill, then move east. If your step north is uphill, then move south. If your step east is downhill, then move west. At each step you are measuring the partial derivative of f(x,y) = z "How much elevation (z) will I gain for a small change in x?"

After enough of these steps, you should reach the bottom of the mountain. The "gradient" in gradient descent is just a vector holding the partial derivatives. Basically a map to tell you which directions will bring you down the mountain for any given step.

![](/images/ik_post/Gradient_descent.gif)

Notice how one of the balls in this gif gets stuck on a little divot on top of the hill. This is called a "local minimum". These can be tricky to escape. There are some methods that help escape these traps like momentum, which carries over between steps and gives you enough "oomph" to roll out of these divots. There are also algorithms like Evolutionary Algorithms which have a random component that helps them "jump" out of these local minimum. For now lets KISS and just focus on gradient descent.

## Implementation of a Robot Arm in 2D

By taking a lot of little steps in the right direction, we can move our robot arm to the target destination!

[Related Video](https://www.youtube.com/watch?v=HCdw4TMdalg&ab_channel=Seabass)

To find the partial derivatives at each angle and calculate the gradient is pretty simple.

1. Start with some θ
2. Find the position of the end of the arm for all the angles
3. Find the difference between actual arm position and target arm position. This is the "Loss" for the current arm configuration. (Loss1)
4. Nudge θ by some small number, d, like 0.00001. This is θ'
5. Find the position of the end of the arm for all angles including this new angle, θ'
6. Find the difference between this new arm position and target arm position. This is Loss2
7. Find (Loss2 - Loss1) / d

This works out to the difference in Loss for a given difference in the angle θ
Or dF/dθ

Computing this for every joint in the arm will give us a gradient. Then all we need to do is subtract this gradient from the vector of arm angles, and it will bring us closer to the target position.
