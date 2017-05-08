---
layout: post
author: Tony
title : Book Club - Land of Lisp Chapters 1-2
date  : 2011-05-30
---

So my friend, and fellow developer, Jeff and I have decided to start a weekly book club. To kick things off, we wanted to start with something fun. With this in mind, we landed on Land of Lisp by Conrad Barski. The book claims to teach Lisp programming through a series of text based games. Sounds pretty cool to me…

The first thing I learned was how to define a global variable. Having always been told to “avoid globals if possible” in most C-based languages, I found this a bit strange. But, in the spirit of keeping my mind open to new convention, let’s define some globals.

```
(defparameter *my-global-var* 42)
```

Apparently the *'s around the variable name are known as “earmuffs”. I like this language already :)

Functions were next up in the reading. However, how to define a function came as no surprise since I did a tiny bit of Scheme programming a few years back.

```
(defun my-first-lisp-function (a b)
    (+ a b))
(my-first-lisp-function 1 1) 
=> 2
```

As one might have assumed, lisp comes with some of its own built in functions. The first built-in function mentioned in the book was the ‘ash’, or arithmetic shift function. This function takes two arguments, a number and the number of bits you’d like to shift that number. For example:

```
(ash 11 -1)
=> 5
```

Ash is looking at the number 11 in binary (1011) and shifting its bits to the right (101), resulting in the new number 5. The book uses this function in the implementation of book’s first game Guess the Number, which uses a binary search to narrow down possibilities until the game hones in on the number the player is thinking about.

Now that we’re getting a little bit familiar with lisp syntax, the book mentioned how to define local variables and local functions. Let’s take a look (Pun intended).

```
(let ((foo 3)
      (bar 4))
    (+ foo bar))
=> 7
```

In lisp, you use the function 'let’ to declare local variables. In my example those variables are foo and bar, which are 3 and 4 respectively. The variables foo and bar are only in scope inside the body of the let function. Neat!

Unsurprisingly, the same can be done with functions. You’ll notice a very similar syntax in the function flet, which is used to define local functions.

```
(flet ((plus-one (n)
           (+ n 1))
       (plus-two (n)
           (+ n 2)))
    (plus-one (plus-two 1)))
=> 4
```

So, now we have a nice little way to define local functions and variables. But, you might be curious about the scope of functions defined inside of flet. For example, can the plus-one function be called inside the plus-two? The answer is no. In order to achieve that, you have to use the labels function. Like so:

```
(labels ((plus-one (n)
            (+ n 1))
         (plus-two (n)
            (+ (plus-one n) 1)))
    (plus-two 2))
=> 4
```

And that’s about as far as I’ve gotten so far. I hope to continue blogging as Jeff and I make our way through this, and hopefully other books.
