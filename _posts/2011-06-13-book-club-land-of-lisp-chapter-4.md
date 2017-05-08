---
layout: post
author: Tony
title : Book Club - Land of Lisp Chapter 4
date  : 2011-06-13
---

Heh, it seems I’ve been reading more than blogging as of late. Anyway, let’s get down to business. This chapter focuses on implementing conditional logic in lisp.

It starts out by pointing out that an empty list () evaluates to false. It is also important to know that an empty list has three equivalences:

```
'()  => NIL
()   => NIL
NIL  => NIL
'NIL => NIL
```

The fact an empty list evaluates to false turns out to be incredibly useful for recursion. In the last post I mentioned the recursive nature of car and cdr, but with conditionals in the mix, we can test when to stop the recursion. The recursion technique I’ve just described is termed “list-eating”. Here’s a little (naive) example of a list-eating recursive function:

```
(defun sum (list-of-num)
    (if list-of-num
        (+ (car list-of-num) (sum (cdr list-of-num)))
        0))

  (sum '(1 2 3 4)) => 10
```

In the code snippet above, you may have noticed I did something new. Well, sort of. I’m sure most people reading this have seen an if statement before, so it’s not all that exciting. However, the author uses this seemingly simple construct to teach a valuable lisp lesson. In lisp, when a function is called, all expressions after the function declaration are evaluated before the function is evaluated. This isn’t the case for “if”. In fact, “if” can only evaluate one expression. This exception to the rule makes “if” the first special form we’ve learned about.

I don’t know about you, but I’d like the ability to evaluate more than one expression. The chapter discusses a few ways to get around this limitation to the “if” special form. The first of which is the progn function. This function essentially lets you hack in a wrapper around your expressions, thus allowing you to evaluate multiple expressions. Check it out:

```
(defparameter *was-executed* 'f)

*was-executed* => F

(defun sum (list-of-num)
    (if list-of-num
         (progn (setf *was-executed* f)
              (+ (car list-of-num) (sum (cdr list-of-num))))
               0))

*was-executed* => T
```

### Do it when this or unless that

Aside from “if”, Lisp offers the “unless” and “when” conditionals. I like these a lot since they read nicely. However, the book notes that they do have one drawback. If the condition does not succeed, nothing is evaluated and nil is returned. Here’s an example of how they might be used:

```
(defun demo-when (list)
    (when (equal (length list) 4)
        (cons "has length of four" list)))

(defun demo-unless (list)
    (unless (null list)
        (cons "is empty" list)))

(demo-when '(0 1 2 3)) => T
(demo-unless '(0 1 2 3)) => NIL
```

### The ultimate conditional

Perhaps the most versatile conditional is cond. Cond allows you to execute multiple things without having to shim in a call to progn, and it allows you to provide an equivalent to the common C-esque control pattern if/else if/else. Of course the syntax is a little different, but I quite like it. Let’s take a peek.

```
(defun conditional (control)
    (cond ((eq control 'if) 'if)
          ((eq control 'ifelse) 'ifelse)
          (t 'fellthrough)))

(conditional 'if) => IF
(conditional 'ifelse) => IFELSE
(conditional 'neither) => FELLTHROUGH
```

While we’re here talking about conditionals, I’d like to take a brief moment to mention logical and/or in lisp. Welp, they work how you think they would.

```
(or 0 0) => 0
(or 0 1) => 1
(or 1 1) => 1
(and 0 0) => 0
(and 1 0) => 0
(and 1 1) => 1
```

It has been brought to my attention that the above may be a tad misleading. Check out this reblog for more info [here](http://n0p.tumblr.com/post/7089691387/book-club-land-of-lisp-chapter-4).

### Equality in Lisp

So up until this point, everything has seemed pretty nice. It couldn’t last forever right? The author closes chapter four with a note about the equality comparator(s) in Lisp. Yes, plural. There are functions named eq, eql, string-equal, char-equal, equal, and =. So, long story short, there are lots of ways to do equality comparisons in Lisp. Most of them are specific to a certain type or fall short when comparing certain things. Thankfully the author included a good rule of thumb that I hope to commit to memory:

1. Use the eq function when comparing symbols
1. Use equal for everything else

Following those two simple rules has kept me out of harms way (so far). The next update may be a bit delayed as I’ll be traveling overseas for nearly three weeks. Although, I did pick up The Little Schemer for some light reading on the plane. Perhaps I’ll do a quick review of when I return. Ciao!

*EDITS:*

* Link to clarification post for Logical and/or
