---
title: "Working title: Fun with Comby and Campaigns"
author: Beyang Liu
authorUrl: https://twitter.com/beyang
publishDate: 2020-05-06T10:00-07:00
tags: [blog]
slug: working-title-fun-with-comby
heroImage: https://about.sourcegraph.com/blog/our-abcs.png
published: true
---

Title: look at headlines from Hacker Noon (https://hackernoon.com/)

<!-- Intro too long and boring --> 
\<Intro\>

<!-- A programmer's job is to automate the tedious. A meta-programmer's job is to automate the tedious parts of programming. There are so many tedious things in programming that every programmer will eventually become a meta-programmer. -->

<!-- One tedious task that pops up frequently in programming is find-and-replace. Examples include: -->

<!-- * Massaging copy-pasted dataset into a particular format -->
<!-- * Updating the callers of deprecated function -->
<!-- * Renaming a misnomer -->
<!-- * Updating a version string -->

<!-- Whatever it is, if you approach the find-and-replace task naively, it will involve you mindlessly typing the same things over and over again, updating each place where the pattern occurs. This is a poor use of your unassisted monkey brain, which is prone to making errors and prefers creative tasks to rote ones. Software should come to the rescue. The problem is that the tools you can use to help execute find-and-replace are not always easy to pick up. -->

<!-- In this blog post, we'll go over a bag of tools to automate find-and-replace tasks in programming. -->

* Tools in your editor
  * String find-and-replace
  * Language-aware refactoring

## Editor find and replace

## Keyboard macros

Keyboard macros are a feature of some editors (e.g., [Emacs](https://www.gnu.org/software/emacs/manual/html_node/emacs/Keyboard-Macros.html), [Vim](https://hackernoon.com/an-intro-to-vim-macros-f690d8c3c3fd), [IntelliJ](https://www.jetbrains.com/help/idea/using-macros-in-the-editor.html)) that let your record keystroke patterns and then replay them later. They are insanely useful if you can express your find-replace operation in a set of keystrokes. Let's take a look at an example:

Say you want to turn an HTML table like this:
```
<table>
  <tr><td>John</td><td>25</td><tr>
  <tr><td>Alice</td><td>24</td></tr>
  ...
  <tr><td>Pat</td><td>31</td></tr>
</table>
```
into a JSON list like this:
```
[
  { "name": "John", "age": 25 },
  { "name": "Alice", "age": 24},
  ...
  { "name": "Pat", "age": 31 },
]
```
To do that with keyboard macros in Emacs, you can type:
```
Ctrl-x (                               # begin recording the macro
Ctrl-s < t d Ctrl-f                    # move curser to 'John'
Ctrl-<space> Ctrl-a Ctrl-w             # delete everything on the line prior to 'John'
" Alt-f "                              # pute double quotes around 'John'
Ctrl-<space> Ctrl-s < t d > Ctrl-w     # delete everything from the end of "John" to '25'
, <space> " Alt-f " <space> ,          # put quotes around '25' and add a comma
Ctrl-a Ctrl-n                          # move cursor to the beginning of the next line
Ctrl-x )                               # finish recording macro
```

Once you've recorded the macro, you can replace it with `C-x e` and you can hold down `e` after that
to apply it repeatedly. Here's it in action:


















Indent. In fact, I used it while writing this blog post, to modify the indentation of the comments
in the above code block to something appropriate.


Examples of tasks that macros are good for:
Translate HTML table to TypeScript map
Updating invocations of an API in a few files
Batch indenting or deindenting
Cases where the pattern isn't apparent / interactive pattern discovery
codemod
grep and sed
Also: ripgrep

Comby
Reference: https://www.sep.com/sep-blog/2019/11/13/challenge-your-favorites/ 
Distributed refactoring tools
Campaigns

This is a good

It could be 

What other tips and tools do you use?
vim for ETL (Keith Amling's workflow)
