---
layout: post
title: "Complaining About Python's Indentation Is Dumb"
date: 2019-11-30
---

One of the more popular criticisms I see against Python is in regard to enforced indentation, here I make the claim that such criticisms are ultimately stupid. To understand the criticism it is important to understand how Python differentiates between blocks. Typically, blocks are denoted by an enclosing pair of curley braces '}' but Python denotes blocks with indentation levels.

```py
def f(x, y):
    # new block
    if x > y:
        # new block
        x *= x
    elif x < y:
        # new block
        x *= y
    return x
```

As you can see, each nested block is denoted by an increasing indentation level. Python will validate these indentation levels using a module named `tabnanny`, an example of its functionality is shown below.
```py
$ cat t.py
def cmp(x, y):
    z = None
    if x > y:
        print("gt")
       z = True
    elif x < y:
        print("lt")
        z = False
    return z

$ python t.py

  File "t.py", line 5
    z = True
           ^
IndentationError: unindent does not match any outer indentation level
```
As you can see, Python verifies that indentation is consistent to know which block a statement is part of. In other languages blocks are, typically, denoted by curly braces.

```c
int main()
{
    /* new block */
    puts("Hello, World!");
}
```

Within these curley braces the indentation level is completely unenforced by the language. I will now outline and discuss various criticisms.

## It forces a developer to use a certain code style

I believe this is the most valid criticism. Yes, to a certain degree you are forced to format your code. However, if you look at style guides for practical programming languages (i.e. not esolangs) it is almost guaranteed that blocks will require an additional indentation level. I have used a multitude of languages from different paradigms and domains but almost all of them had their most popular style guide mandate indentation levels (this is certainly true for imperative languages). You do not lose anything by having enforced indentation, you are almost certainly doing it already. If you are used to imparative languages and choose not to indent at each new bock, please show me your code so I can see how you appropriately format it. Currently there is a rise in popularity for opinionated formatters amongst developers because the merits of having a consistent, _tool enforced_ format has made code bases more consistent. Enforcement of indentation is beneficial as it makes all Python code immediately consistent to a certain degree. For any snippet in any codebase, a developer merely needs to look at the whitespace of each line to know how each line is grouped. I have heard arguments of there being situations where increasing the indentation level is messy. Yes, there are cases where you just want to have code on a single line. It should be first noted that the following is valid:
```py
def f(x, y):
    if x > y: return x
    return y
```

Contrived I know, but you see my point. I don't rely on situations like this however because I believe the general case is more important than a handful of edgecases. Creating edgecases makes code harder to read because now you must remember a number of rules just to save yourself a few lines or characters. Stick to a general rule, consistency is better than a few 'cleaner' chunks of code.

## Indentation makes code move too far horizontally

I will admit, the current PEP8 rule that code should not exceed 80 characters horizontally is unwise (this is being changed). However, in many style guides for many languages, there is a general rule: if you exceed 4 indentation levels you should probably define a function. If you find yourself squashed up against the character limit you almost definitely need to reconsider your applicaion structure. If you still find yourself stuck at the character limit and Python developers are yelling at you, you would experience the same thing in any other language (perhaps you should consider a different language for a more elegant solution).

## The code doesn't even run if indentation isn't consistent

Indentation is not a strenuous task that requires you to hop around and fix imports _cough cough Golang cough_. Your editor should have the ability to autoindent code for you, if not then I have to seriously question your setup.

## It breaks if you mix tabs and spaces

If you are mixing tabs and spaces to indent code then you are not adhering to any reasonable style guide. Mixing indentation makes your code immediately inconsistent both from content and display (tabs do not have a fixed column width). If you are not having a consistent width for indentation levels then I can't say I have much hope for your code to ever be much pleasure to work with.


# Why Indentation Is Important Stylistically

Consider the two snippets below.

```c
/* No indentation */
int main() {
int y;
int x = get_int();
int i = 0;

if (x > 5) {
y = x * 2;
for (int z = 0; z < x; z++) {
x *= (z+1);
}
} else {
y = x;
}

while (x++ < y){
i++;
}
printf("%d\n", i);
}
```

```c
/* Indentation */
int main() {
    int y;
    int x = get_int();
    int i = 0;

    if (x > 5) {
        y = x * 2;
        for (int z = 0; z < x; z++) {
            x *= (z+1);
        }
    } else {
        y = x;
    }

    while (x++ < y){
        i++;
    }
    printf("%d\n", i);
}
```

You can immediately see various program structures. The ability to glance at code and immediately pick apart various control structures: loops, predicates, functions, etc. is extremely important for the code readability. Forcing this indentation is not a curse, it is an improvement.


