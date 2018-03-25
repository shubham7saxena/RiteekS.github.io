---
layout: post
title: Importance of coding convention and its practice
date: 2017-07-10 23:30:09
comments: true
categories:
- Computer Science 
tags:
    - Programming
---

Most of the people think that why should any one care about the convention if the program is able to deliver the product with all the required features. What is in name? There is very famous quote

> A rose by any other name would smell a sweet.

 but I am sorry to say that name matter a lot in programming and coding. It is very certain that at corporate level one will have to work on the code written by other and if there is no convention followed in the code then it may be impossible to edit that code, Leave for others case, after some time the who wrote the code will not be able to understand and edit the code. I have gone through this situation. In the college I didn’t use to care about coding conventions and project structures. But I think everyone should make this practice of following conventions and project structures. If today I see the code of the projects that I wrote in college then I myself will not be able to understand it.

If you want your code to be scalable and maintable then you must follow the conventions. It is not only the criteria for scalable and maintainable code but it is very basic requirement for the same.

Code is the best documentation for a software because any other document and comments can become outdated quickly, but code will always tell the truth. Every effort for the conventions and structure pays both short time and long time benefits. Following are some tips for doing the good coding practices.

### Give Meaningful Names
Naming should be given very carefully. The name should implies what it does or for what is it use for? If this is followed in the code then there is no need of comment. And a good code should not contains any comment if it does then you should do rm — rf :stuck_out_tongue_winking_eye:

### Avoid Similar Names
Using similar names is a worse practice. Because it will be very hard to spot the differences for example having two variables coordinate and coordinates, has every same character except last one. This kind of differences are very hard to spot and are even harder to find during the code review.

### Prefer descriptive name over short form
Name should be able to describe its context, if name is going longer for the same reason then there is no problem, use the longer name. For example use gameOfLife instead of gol.

### Use Consistent Naming
This is another best coding practice. Try not to use the synonyms for similar methods like kill(), finish().

### Make use of common verbs
One should use is, has, can etc in the variable name. Boolean variable with ‘is’, ‘has’ are more readable.

The naming nomenclature and conventions are different in different languages and we should follow that because you should always keep in mind that you are not writing the code for yourself rather for others. Hence to make the code readable to others there should be one convention and nomenclature (at least within a team)which should be followed by every one.



