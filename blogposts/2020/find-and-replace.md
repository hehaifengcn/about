---
title: "Find and Replace: a Programmer's Odyssey"
author: Beyang Liu
authorUrl: https://twitter.com/beyang
publishDate: 2020-05-06T10:00-07:00
tags: [blog]
slug: working-title-fun-with-comby
heroImage: https://about.sourcegraph.com/blog/our-abcs.png
published: true
---

So much of what we do as programmers boils down to automating the tedium out of our work. We'd like
to spend our lives focusing on the design of beautiful abstractions and algorithms, but before we
can get to that, we have to do the dirty work—and do it quickly.

Whether you are a newbie or a seasoned engineer, chances are that you struggle with day-to-day tasks
that seem to take longer than they should. There is, of course, no overarching theory or single key
insight, just an ever-growing bag of tools and tricks one acquires over time.

This post is the first in a series that will cover some of these tools and tricks. <!-- TODO:
mention it might be standalone, but it might be the first of many --> It's intended for a technical
audience of all levels of experience. Inexperienced programmers will get a whirlwind tour of things
they can use to level-up their engineering productivity, and experienced programmers will hopefully
encounter a few new tools and ways of thinking about things.

So, let's talk about find-and-replace.

## Find and replace

At the most basic level, you have literal string substitution, which means you look for a literal
string and replace it with another literal string. Almost every editor, from IntelliJ to Microsoft
Word, has a find-replace feature.

<!-- Screenshot of VS Code find-replace menu -->

Programmers, however, quickly find that literal find-and-replace falls short. This is because there
are many scenarios in which programmers want to apply a general change pattern in many
locations. Refactoring, renaming, updating callers of a deprecated API, and transforming data from
one format to another are all cases that call for a find-and-replace tool that's more than just
literal string substitution.

## Regular expressions

The most commonly used pattern matching language is Regular Expressions (commonly abbreivated as regex
or regexp).

<!-- historical context about regex and why they were invented -->

With regex, you can do things like this:

* TODO: list of English language descriptions and corresponding regex


I say "most commonly used", but there are many that struggle to use them. I knew of one engineer who had years of experience who couldn't even construct a simple regex. This is not a dig at him, because it's true that regex are generally not super user friendly.

Use a regex visualizer. Any of these will do:





* String literal find-and-replace
  * Most basic find/replace
  * In every editor
* Keyboard macros
* Regular expressions


















<!-- ## Keyboard macros -->

<!-- Keyboard macros are a feature of some editors (e.g., [Emacs](https://www.gnu.org/software/emacs/manual/html_node/emacs/Keyboard-Macros.html), [Vim](https://hackernoon.com/an-intro-to-vim-macros-f690d8c3c3fd), [IntelliJ](https://www.jetbrains.com/help/idea/using-macros-in-the-editor.html)) that let your record keystroke patterns and then replay them later. They are insanely useful if you can express your find-replace operation in a set of keystrokes. Let's take a look at an example: -->

<!-- Say you want to turn an HTML table like this: -->
<!-- ``` -->
<!-- <table> -->
<!--   <tr><td>John</td><td>25</td><tr> -->
<!--   <tr><td>Alice</td><td>24</td></tr> -->
<!--   ... -->
<!--   <tr><td>Pat</td><td>31</td></tr> -->
<!-- </table> -->
<!-- ``` -->
<!-- into a JSON list like this: -->
<!-- ``` -->
<!-- [ -->
<!--   { "name": "John", "age": 25 }, -->
<!--   { "name": "Alice", "age": 24}, -->
<!--   ... -->
<!--   { "name": "Pat", "age": 31 }, -->
<!-- ] -->
<!-- ``` -->
<!-- To do that with keyboard macros in Emacs, you can type: -->
<!-- ``` -->
<!-- Ctrl-x (                             # begin recording the macro -->
<!-- Ctrl-s < t d Ctrl-f                  # move curser to 'John' -->
<!-- Ctrl-<space> Ctrl-a Ctrl-w           # delete everything on the line prior to 'John' -->
<!-- " Alt-f "                            # pute double quotes around 'John' -->
<!-- Ctrl-<space> Ctrl-s < t d > Ctrl-w   # delete everything from the end of "John" to '25' -->
<!-- , <space> " Alt-f " <space> ,        # put quotes around '25' and add a comma -->
<!-- Ctrl-a Ctrl-n                        # move cursor to the beginning of the next line -->
<!-- Ctrl-x )                             # finish recording macro -->
<!-- ``` -->

<!-- Once you've recorded the macro, you can replace it with `C-x e` and you can hold down `e` after that -->
<!-- to apply it repeatedly. Here's it in action: -->

<!-- ![Emacs keyboard macros animation](/blog/find-replace/macro.gif) -->

<!-- Once you start using them, you'll find yourself reaching for keyboard macros for many tasks. In -->
<!-- fact, I used them in writing up this blog post, to modify the indentation of the comments in the -->
<!-- codeblock above. -->



<!-- ## grep and sed -->

<!-- Also: ripgrep -->

<!-- ## Comby -->

<!-- Reference: https://www.sep.com/sep-blog/2019/11/13/challenge-your-favorites/  -->


<!-- ## codemod -->

<!-- ## Campaigns -->



<!-- Outro: What other tips and tools do you use? -->
