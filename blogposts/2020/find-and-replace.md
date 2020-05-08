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
Word, has a find-replace feature for string literals.

<!-- Screenshot of VS Code find-replace menu -->

Programmers, however, quickly find that literal find-and-replace falls short. This is because there
are many scenarios in which you want to apply a general change pattern in many
locations. Refactoring, renaming, updating callers of a deprecated API, and transforming data from
one format to another are all cases that call for a find-and-replace tool that's more than just
literal string substitution.

## Regular expressions

The most commonly used pattern matching language is Regular Expressions (commonly abbreviated as
"regex").[^1]

<!-- historical context about regex and why they were invented -->



With regexes, you can do things like this:

| Description                                                  | Regex      | Match         | Does not match |
|--------------------------------------------------------------|------------|---------------|----------------|
| Find all symbol names starting with "http"                   | `foo\w*`   | fooBar        | barFoo         |
| Find all character strings between two sets of double quotes | `"[^"]"`   | "hello world" | 'hello world'  |
| Find all references to fields of a certain variable          | `\w+\.\w+` | base.Path     | basePath       |

Here's a short primer on the syntax:

* `.` matches any alphanumeric character. To match a period, use `\.`
* `[`...`]` matches a single character in a character set. For example `[ABC]` matches either "A",
  "B", or "C". `[A-Za-z]` matches any upper- or lower-case letter.
* `\w` mataches any alphanumeric character or underscore (it is equivalent to `[A-Za-z0-9_]`. `\W` matches any non-alphanumeric character.
* `*` means "zero or more of the preceding character".
* `+` means "one or more of the preceding character".
* `?` means "zero or one of the preceding character".
* `\s` matches a single character of whitespace (e.g., spaces or tabs). `\S` matches any non-whitespace character. `\b` matches a word boundary.

Most regex implementations also support a way to *capture* parts of the match, which can then be
referenced in a replacement pattern. For example:

| Description | Reverse the order of arguments in a function call |
|-|-|
| Regex | `myFunc\((\w+), (\w+)\)` |
| Replacement pattern | `myFunc(\2, \1)` |
| Input &rarr; Output | `myFunc(foo, bar)` &rarr; `myFunc(bar, foo)` |

Note that the replacement pattern syntax lacks a standard, and different implementations may use
different notation.

<!-- TODO: change code example to reference Sourcegraph repository, add links to Sourcegraph. Encourage people to try it live. -->

Now, regex is only the syntax. To use them, you have to use a tool that implements them. The most
commonly used such tools are the Unix command-line programs, `grep` and `sed`.

## grep and sed

`grep`[^2] is a program that scans text files line-by-line and prints each line that matches a specified
pattern. `sed`[^3] is a tool that matches and transforms text. Both are extremely versatile and useful
tools to have in your programmer's toolkit.

Here's a motivating example. Suppose you add an additional parameter to some function and now need
to update all callers of that function to pass some default value for the extra argument. One
approach is to use your editor's find-and-replace feature and manually update all call sites.[\4]
This, however, is quite tedious. Here's a one-liner with `grep` and `sed` that accomplishes this:

```
grep -lRE 'errorutil\.Handler' | xargs sed -i -E 's/Handler\(([A-Za-z0-9_\.]+)\)/Handler(\1, "default value")/'
```

It's not the clearest thing in the world, so let's break it down:

| Part | Description |
|-|-|
| `grep` | We're using `grep` to generate a list of filenames that contain the specified pattern. We'll then feed these to `sed` to do the actual replacing. |
| `-lRE` | The `-l` flag tells `grep` to print only filenames. The `-R` flag tells `grep` to do a recursive search in the current directory. The `-E` flag tells grep to use the extended regexp syntax, which we use throughout this entire post. |
| `'errorutil\.Handler'` | This is the regex pattern that selects for matches like `errorutil.Handler(serveRepoBadge)`. Note that our regex doesn't have to match the entire expression we want to replace, because the purpose here is only to filter down to a small enough set of filenames to pass to `sed`. |
| `|` | This is a Unix pipe, which forwards the standard output of the previous command to the standard input of the following command. |
| `xargs` | This is a program that wraps other commands to map standard input to command line arguments. `sed` takes filenames as command line arguments, and this is necessary to map the output of `grep` to command line arguments to `sed`. |
| `sed` | We use `sed` to transform the contents of each file passed to it following the pattern we specify. |
| `-i` | This flag tells `sed` to modify files "in place", rather than printing the transformed contents to standard output. |
| `-E` | This flag tells `sed` to used extended regexp syntax. |
| `s`<br>`/`<br>`Handler\(([A-Za-z0-9_\.]+)\)`<br>`/`<br>`Handler(\1, "default value")`<br>`/` | This specifies the replacement pattern and is a bit of a doozy, so let's break it down even further. This is actually an expression in the `sed` language. `s` is the "substitute" command. The character immediately after `s` specifies the delimiter that will separate arguments to `s`. (In this case, it is `/`, but we can make it anything we want so long as we're consistent.) The first argument, `Handler\(([A-Za-z0-9_\.]+)\)`, is a regular expression with a matching group to capture the argument to the function call. The second argument, `Handler(\1, "default value")`, is a replacement pattern, which references the first and only regex capture with `\1`. |

If all this is clear as mud, don't worry—you're not alone. `grep` and `sed` are powerful tools, but
they're not super beginner-friendly.[^5] [^6]

[ripgrep (`rg`)](https://github.com/BurntSushi/ripgrep) is a newer alternative to `grep` that is
often faster and has more user-friendly defaults (i.e., you don't need to remember to set flags like
`-lRE` to get the behavior you want). Here's the one-liner above rewritten with ripgrep, instead of grep:

```
rg -l 'errorutil\.Handler' | xargs sed -i -E 's/Handler\(([A-Za-z0-9_\.]+)\)/Handler(\1, "default value")/'
```

Very soon, we'll cover more alternatives that address common pain points.

## Keyboard macros

One pain point of using `grep` and `sed` and regular expressions in general is writing a regex is difficult. The syntax is not very readable, so you are prone to make mistakes and it may take multiple attempts with a fair amount of debugging to arrive at the right regular expression.

Keyboard macros are a feature of some editors (e.g., [Emacs](https://www.gnu.org/software/emacs/manual/html_node/emacs/Keyboard-Macros.html), [Vim](https://hackernoon.com/an-intro-to-vim-macros-f690d8c3c3fd), [IntelliJ](https://www.jetbrains.com/help/idea/using-macros-in-the-editor.html)) that let you record keystrokes and replay them later. If you can describe the change you'd like to make in a set of keystrokes, a keyboard macro can be much easier to execute than `grep` and `sed`.

Let's revisit our earlier example of wanting to add an additional parameter to `errorutil.Handler`. In my editor (Emacs) I can execute the change I'd like to make in the following keystrokes:
```
Ctrl-x (                   # begin recording the macro
Ctrl-s errorutil.Handler(  # search for the pattern
Ctrl-Alt-n Ctrl-b          # jump to one character before the end parens
, "default<space>value"    # add the default value for the new argument
Ctrl-e                     # move cursor to end of the line
Ctrl-x )                   # finish recording the macro
```

Once you've recorded the macro, you can replay your keystrokes with `C-x e` and you can hold down `e` after that to apply it repeatedly.

Here's another cool example. Say you want to turn an HTML table like this:
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
Ctrl-x (                             # begin recording the macro
Ctrl-s < t d Ctrl-f                  # move curser to 'John'
Ctrl-<space> Ctrl-a Ctrl-w           # delete everything on the line prior to 'John'
" Alt-f "                            # pute double quotes around 'John'
Ctrl-<space> Ctrl-s < t d > Ctrl-w   # delete everything from the end of "John" to '25'
, <space> " Alt-f " <space> ,        # put quotes around '25' and add a comma
Ctrl-a Ctrl-n                        # move cursor to the beginning of the next line
Ctrl-x )                             # finish recording macro
```

And here's it in action:

![Emacs keyboard macros animation](/blog/find-replace/macro.gif)

As powerful as keyboard macros are, they are only supported in some editors (e.g., [Emacs](https://www.gnu.org/software/emacs/manual/html_node/emacs/Keyboard-Macros.html), [Vim](https://hackernoon.com/an-intro-to-vim-macros-f690d8c3c3fd), [IntelliJ](https://www.jetbrains.com/help/idea/using-macros-in-the-editor.html)). They also are difficult to scale to changes across many files. Luckily, there's a new tool that addresses many of the common pains of regular expressions while also scaling to changes across many files, even those that may be outside your editor workspace.


## Semantic tools

TODO

* Editor-based semantic refactoring (IntelliJ, LSP)
* codemod

## Comby

Comby is a new tool for changing code that introduces a simple syntax for matching patterns commonly found in source code. It aims to be both more powerful and more user-friendly than regex.

Here is a one-liner that handles the earlier use case of adding an extra argument to `errorutil.Handler`:

```
rg -l errorutil | xargs comby -in-place 'errorutil.Handler(:[1])' 'errorutil.Handler([1], "default value")'
```

| Part | Description |
|-|-|
| `rg -l errorutil` | Use ripgrep to print names of all code files that contain "errorutil" |
| `| xargs` | Pipe the filenames to `comby` |
| `comby -in-place` | Edit the files in-place with the `comby` CLI |
| `errorutil.Handler(:[1])` | The match pattern, which sub-matches the argument using the Comby [hole syntax](https://comby.dev/#basic-usage) |
| `errorutil.Handler([1], "default value")` | The replace pattern, which references the sub-matched hole |

This is already much more user-friendly than the regex pattern we wrote when using `sed`.

It is also more powerful. To see how, let's revisit the `errorutil.Handler` example by considering the possibility that the following call site exists:

```
errorutil.Handler(someOtherFunction(blah))
```

This call site would not be selected by our earlier regular expression, which doesn't handle the nested parens. We could try to update the regex to account for nested parens, but it is difficult to do so. The best I could come up with is this:

```
TODO
```

Even then, there are many other call site examples that would break the new regex. For example, what if we had one more layer of nesting or the first argument was a string constant that contained parentheses?

In fact, these difficulties lead to a more general limitation of regular expressions: they lack the expressive power to define nestable "bookend" patterns of the form `<delimiter>stuff<delimiter>`. Such patterns common occur in code. For example, string constants, nested function calls, nested control blocks, array literals, and struct literals are all syntactical constructs that cannot be defined in a general way using regular expressions.

With Comby, however, those patterns are super simple to define.

>>> Table with a bunch of comby examples
>>> Also include the HTML example from earlier



[^1]: I say "most commonly used", but fluency with regex is by no means ubiquitous. I knew one
    engineer who had years of experience and was a contributor to several prominent open-source
    projects, but regex was a foreign language to him.

[^2]: "grep" stands for "global regular expression print".
[^3]: "sed" stands for "stream editor"
[^4]: Many editors support regex search, but not all support replacing with regex matching groups
    (or it is non-obvious how to do so). Even if the editor supports both, you may want to update
    files that aren't currently open in your editor's workspace or context.
[^5]: Another complicating factor is that there are multiple implementations of `grep` and `sed`
    that have slightly different behavior and are defaults on different systems. macOS by default
    uses the BSD variants of `grep` and `sed` while Linux uses the GNU variants. The GNU and BSD
    variants don't support the same sets of flags and positional arguments, so `grep` and `sed`
    invocations that work on Linux often break on macOS or vice versa. You can get around this by
    installing the non-default variant on either system, but this, of course, adds complexity.
[^6]: Two other Unix commands that are often used in conjunction with `grep` and `sed` are `find`
    and `awk`. For brevity's sake, we don't go into them here.

<center>~ 30 ~</center>




Use a regex visualizer. Any of these will do:

Difficult to write, virtually impossible to read.

## Comby

## Campaigns

## Appendix

### codemod

### ripgrep

...Maybe someone in the ripgrep community will add support for Comby :)

~~~~

<!-- Diss regex in the intro to Comby section -->

<!-- * String literal find-and-replace -->
<!--   * Most basic find/replace -->
<!--   * In every editor -->
<!-- * Keyboard macros -->
<!-- * Regular expressions -->


















<!-- ## Keyboard macros -->




<!-- ## grep and sed -->

<!-- Also: ripgrep -->

<!-- ## Comby -->

<!-- Reference: https://www.sep.com/sep-blog/2019/11/13/challenge-your-favorites/  -->


<!-- ## codemod -->

<!-- ## Campaigns -->



<!-- Outro: What other tips and tools do you use? -->
