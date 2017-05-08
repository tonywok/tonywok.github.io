---
layout: post
author: Tony
title : Book Club - Land of Lisp Chapter 3
date  : 2011-06-06
---

Another week has gone by, and with that comes another Land of Lisp reading assignment. Since this chapter contained a bit more information than the first two, I’ve decided to split chapters 3 and 4 into two separate posts.

Chapter three served as an introduction to the syntax of lisp. This chapter was somewhat familiar to me since, like I said before, I’ve done a little bit a of scheme. However, it was a nice refresher.

The chapter starts by introducing lists, the quintessential piece of syntax in lisp. Lists can contain pretty much anything including symbols, numbers, and strings.

### Symbols

In lisp, symbols represent data and are passed around prefixed with a single quote so that lisp knows not to attempt to execute them. Also, be sure to keep in mind that symbols are case insensitive. For example:

```
(eq 'some-symbol 'SoMe-SyMbOl)
=> T
```

### Numbers

Lisp handles numbers in a pretty interesting way. It supports both integers and floating point numbers, but the way they are created is very different than what someone coming from a C-based language might be used to. Rather than statically declaring a data type, lisp will infer what you mean based on how you use the number. For example:

```
(+ 1.0 1)
=> 2.0
(+ 1 1)
=> 2
```

Notice that the result was a floating point number in first example where we used a floating point number in our computation. Another interesting feature of lisp computation is how it handles the division of numbers.

```
(/ 1 4)
=> 1/4
(/ 1.0 4)
=> 0.25
```

Pretty cool eh? If you notice, again, the lisp compiler infers that we want a floating point result when we use floating points in our computation. However, in the case of integer division, you may have noticed something odd. No longer will you have to deal with 0.6666[…]7, you can just use rational numbers until the very end of your computation. Depending on how you write your code, I could see this greatly reducing rounding error.
Strings

Like any other programming language, lisp provides a way to handle strings. In lisp, strings are wrapped with double quotes. Unsurprisingly, if you would like to create a string containing double quotes, you are required to escape them with a backslash.
Modes of Operation

During my brief stint programming in Scheme, I was very confused about why some things were prefixed with a single quote. I learned how to use use and write code, but I guess I just didn’t internalize what was actually going on. Lisp (and Scheme) have 2 modes of operation: Data mode and Programming mode. In programming mode, things operate exactly as you’d expect – Every piece of code is executed. Sometimes this isn’t what you want. It would be nice to be able to pass around pieces of code and not have them executed, or at least delay their execution. To accomplish this, lisp enters data mode. In order to enter data mode, code is prefixed with a single quote.

```
'(+ 1 2)
=> (+ 1 2)
```

In the previous example, I prefixed the + function with a single quote. If I type this in the clisp REPL, it returns the function (+ 1 2) instead of 3, as you may have expected.
Essential List Functions

```
(defparameter *test* '(1 2 3 4))

;; cons example
(cons *test* '(I declare a thumb war)
=> ? 

;; car example
(car *test*)
=> ?

;; cdr example
(cdr *test*)
=> ?
```

### Cons

The cons function takes two things and wraps them in a list. In the text, the author notes that a list is made up of cons cells. A cons cell is simply an item in a list. That being said, a list is just a bunch of cons cells glued together using the cons function. For example, the following are all the same:

```
'(1 2 3 4)
(list '1 '2 '3 '4)
(cons 1 (cons 2 (cons 3 (cons 4 ()))))
```

In our cons example above, we’d like to wrap our *test* list in a list alongside ’(I declare a thumb war). The result would then be:

```
'((1 2 3 4) I DECLARE A THUMB WAR)
```

### Car

The car function returns the first item in a list. So, if we look at our car example above, it will return 1, the first item in our list.
Cdr

The cdr function does the inverse of car, or rather, it returns the remaining list minus the first item. So, if we look at our cdr example, we can deduce that it will return (2 3 4). With these two tools at our arsenal, it is pretty obvious that we’ll be taking advantage of a lot of recursion techniques.

The car and cdr functions can be combined in a number of ways to produce a shorthand syntax to interact with lists. Some of these examples include cddr, caddr, cadadr, and so on.

That’s all for now. I’ll have my recap of chapter 4 posted within the next few days.

*Update:* Oh by the way, I’ve been posting code here: https://github.com/tonywok/land_of_lisp

