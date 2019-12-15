---
layout: post
title: "C is archaic"
date: 2019-12-15
---

C has existed since the 70s as a derivative of B. The influence of C should not be understated, it has been a key component of major developments in the tech world (e.g. MINIX). In the modern era, C is wildly outdated compared to its modern counterparts. We no longer have to worry about manual memory collection in unconstrained environments with garbage collected environments such as the JVM or languages with automatic memory management such as C++'s smart pointers.

## The Type System

The type system in C is completely eclipsed with modern languages. The ability to cast `void*` into any object is powerful but is a source of never ending bugs as there is no guarantee to the integrity of the cast (the same is true for C-style casts in general). C has no legitimate support for generic functions, `_Generic` was introduced in C11 but is only capable of specialisation. There are features to have a variable number of arguments beyond the absolute disgrace that is `va_list`. `va_list` is every strongly typed evangelists nightmare, it simply interprets whatever is on the stack (assuming your compiler is adhering to the cdecl calling convention) as whichever type you wish. This has clear issues as evidenced by the well known format string vulnerability.

```c
int main(int argc, char** argv) {
    printf(argv[1]);
}

/*
$ gcc p.c
$ ./a.out "%x"
798927b8
*/
```

We are able to achieve a primitive read and write using the various format specifiers. Below is an example of how to have a variable number of arguments.

```c
void print_ints(int n_args, ...) {
    va_list xs;
    va_start(xs, n_args);

    for (int i = 0; i < n_args; i++) {
        int x = va_arg(xs, int);
        printf("%d\n", x);
    }
}

int main(void) {
    print_ints(5, 1, 2, 3, 4, 5);
}
```

Observe how _insane_ this is. To peel arguments from this `va_list` you need to specify the type which then casts _part of the stack_ (as part of the cdecl calling convention). Also, notice the `int main(void)` declaration, leaving the parentheses empty allows the function to take _an arbitrary number of arguments_.

```c
void foo() {
}

int main(void) {
    foo(1, 2, 3, 4);
}
```

Will compile _without_ any warnings or errors, even when compiled with `gcc test.c -Werror -Wall -Wextra -pedantic`. I cannot understand why this is perfectly valid, _surely_ this is grounds for at least a warning?!

## Error Handling

The way C handles errors is quite honestly the worst possible way to ever do it, _never_ do what C does. I understand that this was an original part of C that cannot be changed but this is a discussion on why C is _archaic_ not a discussion as to why it was bad for its time. Generally, errors are returned in the form of an integer status code (typically 0 for success with negative values denoting errors). This form of error handling is completely scuffed with the definition of each error contained within `errno.h` (accessible with `strerror` from `string.h`). Modern error handling exists in the form of exceptions, value-error pairs, and Maybe's. Exceptions are their own beast but are preferrable to status codes since they allow for cleaner error recovery (and can carry information with them), this form of error handling can be found in: Python, C++, Java, and many others. Value-error pairs are a weaker version of Maybe's, this type of error handling can be found in Golang. Maybe's are mostly used in functional languages (they couple extremely well with pattern matching with sum types), they are the cleanest way to handle errors in my opinion (implementations of this exist as optional types and `Maybe` in Haskell).

```c
int main(void) {
    int fd;
    if ((fd = open("nonexistant", O_RDONLY)) < 0) {
        fprintf(stderr, "err: %s\n", strerror(errno));
    }
    close(fd);
}

/*
$ gcc fail.c
$ ./a.out
err: No such file or directory
*/
```

It should be noted that not _only_ does C just throw around status codes it will store the current status in a global variable named `errno` (however this is thread-local in C11). To access it, the previously mentioned functions can be used (it should be noted that `perror` will resolve the error string and output to `stderr` accordingly).

C will quite happily return `NULL` as an error from any function which is to return a pointer. `NULL` is inherently garbage and exists as a stain in modern software causing headaches from null-pointer dereferences to constantly having to check if a pointer is `NULL` without any compiler support. Modern languages that use the aforementioned exceptions and Maybe's are able to completely side-step `NULL`s (and allow for compiler assistance in error handling).

## Memory Management

Firstly, let me say that manual memory management is not terrible, it allows for careful control over your application which is extremely important in constrained environments (.e.g. embedded or high performance). Typically, we use an implementation of `malloc` then free accordinly with `free`. C has absolutely zero memory safety in the face of modern languages like Rust which are able to safely control memory access. Furthermore, we can isolate areas where we invoke "unsafe" behaviour which allows for bug isolation. C has no ability to automatically free objects once they go out of scope which allows for fiendishly hard to fix memory leaks (though tools do exist like `ASan` and `valgrind`).

```c
void foo(void) {
    int* x = malloc(sizeof *x);
}

int main(void) {
    foo();
    ...
}
```

Notice, the resource `x` is never reaped. While this example is obvious, in a large codebase this can quickly become an extremely time consuming task. Furthermore, C has no ability to mark a pointer as invalid which allows for use-after-free vulnerabilities (aka UAF) and double-free vulnerabilities.

```c
int main(void) {
    int* x = alloc_arr(10);
    consuming_func(x);
    consuming_func(x); /* error */
}
```

Again, this is an obvious example but is finicky to track down in a more complicated codebase.

In an era with increased resources and memory we should be reliant on our tooling to manage them safely but with C that is not the case. It isn't a surprise that memory related bugs are the most common  (as well as the most common source of vulnerabilities). C++ has automatic pointers which destruct a resource once its lifetime ends but still fails to protect against improper usage of free'd pointers (though double frees are fine). Other forms of automatic memory management exist but the most common is garbage collection. Garbage collection is incredibly useful for where you have an unconstrained environment but is typically unpredictable (i.e. inappropriate for highly controlled environments).

A nice error manifested itself a few weeks ago which I think demonstrate how unsafe C is to use.

```c
KYU_FUNC _value_wrapper_obj_ctor(value_wrapper_obj** self, int value) {
    struct _method_table_ent entries[] = {
        {.n_frequency=0, .name="print_value", .method=_value_wrapper_obj_print_value},
        {.n_frequency=0, .name="set_value", .method=_value_wrapper_obj_set_value},
        {.n_frequency=0, .name=NULL, .method=NULL}
    };
    method_table_t* methods = new_method_table(entries);

    (*self)->methods = methods;
    (*self)->value = value;
    return Success;
}
```

Can you see the mistake here? The mistake only manifested itself after an hour once certain conditions were in place (if you want a hint, it was after enough function calls). Spoilers! that struct is allocated on the stack which is then overwritten after enough calls. The fix was to add a `static` type qualifier to `entries[]`. The compiler gave no warnings and left me completely on my own, in a modern language with proper memroy safety this would have been flagged immediately.

## Data Structures

The standard data structures available are absolutely horrific, in order to use a data structure you are forced to implement it yourself (I will touch on dependencies later). What I mean by this is that if you wish to have a hashtable of some sort, you will have to create a data structure _specifically_ for your problem (including the key and value types unless you want to engage in macros). Why is this a problem? You are forced to re-invent the wheel for _all_ your projects which means you are dealing with different APIs between codebases for the same structure and may introduce bugs as your implementation is nonstandard. Currently I am writing an emulator and to have some associative array structure I have to implement a map from scratch, a fantastic learning experience but I really don't want to implement a map, I just want to use one.

Bugs introduced when developing your own data structures relate to the aforementioned memory issues and confusing semantic behaviour of C. In modern languages you can expect a wealth of algorithms and data structures to use out of the box. God forbid you wish to use strings in C, strings are horrific to work with, I suggest you check out [this rant](https://github.com/mpv-player/mpv/commit/1e70e82baa9193f6f027338b0fab0f5078971fbe). Strings in C are error prone due to stack allocation and the need to constantly reallocate memory around them. Concatenation involves reallocating a buffer to avoid buffer or heap overflows (`strcat`, `strdup` and their size-restricted variants).

## Dependencies

Re-inventing the wheel violates one of the key principles of software engineering: someone has probably solved your problem better than you. Dependencies are fantastic when used appropriately, C however, does not lend itself to dependencies. C dependencies are managed by including a header file which contains declarations (or definitions) of symbols in other C files. To understand why this is horrific, `#include <stdlib.h>` is bad, it literally _pastes_ the content of `stdlib.h` into the source file, it removes all notion of namespaces since all symbols are in the global namespace which leads developers to add symbol prefixes like `sigma16_init_vm`. It should also be noted that C has no dependency manager, all you can do is copy and paste a file from somewhere else then slap a `#include` in your source file. C has an obsession with the global scope, from polluting the namespace with your copy-and-pasted dependencies, `errno`, and environment variables. Globals result in extremely weak guarantees when reasoning about code which inherently makes C codebases difficult to understand. Back to dependencies, because includes are weak and there is no dependency manager, you are _forced_ to roll your own data structures at times because there is a complete lack of standards.

Rigid programming like this only encourages symbol collisions and confusing name resolution (e.g. what header does `O_RDONLY` come from?). Modern languages are able to effortlessly beat this with namespaces and even more modern languages have dependency managers like Python with `pip` (although that does have its own limitations).

## Environment Variables

Some functions within C can change behaviour based on environment variables, as you can imagine this is completely asinine. The rant mentioned earlier describes how much of a hack they are.

## Macros

Ever worked with a large C codebase before? Going to guess you have, and you've witnessed the sheer volume of esoteric macros. Macros are similar to dependencies in that they are expanded into the raw source. C macros aren't _that_ bad but they've certainly been improved in languages which have proper generics or more powerful macros like Rust. However, definitions that only expand to a value are horrific, they can cause numerous subtle errors since what you believe is a name is actually a value when expanded!

```c
typedef enum states_ = {
    NORMAL,
    DEBUG,
    EXPERIMENTAL
} States;

/*
$ gcc -DDEBUG defn.c
<command-line>: error: expected identifier before numeric constant
defn.c:3:5: note: in expansion of macro 'DEBUG'
    3 |     DEBUG,
      |     ^~~~~
*/
```

I get it, I get it, "you should know what `-DDEBUG` does" but this is still an absolute headache to fix since _something_ has to be renamed and who knows how many references there are. This is an example of global behaviour being a nightmare again.

## Concurrency

Concurrency in C is an absolute nightmare because of the lack in memory safety. Again, modern languages like Rust allow for safer concurrency by allowing you to reason about memory. In other languages you have other access to threading like goroutines or green threads which allow for _significantly_ more powerful threading and have a greater focus around concurrency, versus simply giving you access to a system level API.

## Conclusion

Understand that I wrote this not to insult C as a language, it has done tremendously, but it is seriously misguided to be using it if you can avoid it. Yes, C is still standard in several environments like embedded systems but that doesn't mean it's a good language for the application (just because you're close to hardware doesn't mean you can't have guarantees relating to higher level behaviour). You should be able to rely on your compiler or implementation to _help you_ write safe and secure code instead of blindly wandering around until you step over a mine.
