---
layout: post
date: 2021-10-10
title: Laziness in C
excerpt: Achieving lazy evaluation in C by unnecessary complex means
---

## Strict evaluation

The C language has strict semantics for most of its expressions. That means pretty much any complex expression gets evaluated first and then the evaluated result is used as the value of the expression. For example, in the following snippet the function call gets evaluated even though its result is never used:

### example1.h:
```c
#include <stdio.h>

int a_or_b(int a, int b) {
    return a > 0 ? a : b;
}

int get_a() {
    puts(__func__);
    return 1;
}

int get_b() {
    puts(__func__);
    /* some expensive computation */
    return 2;
}

int main() {
    a_or_b(get_a(), get_b());
    /*
     * will print
        get_a
        get_b
     * in unspecified order
     */
}
```

Since all the functions here are computed in runtime and have external linkage, there is no way for compiler to know whether it is legitimate to remove a call from an expression.

There are languages which have lazy semantics by default and we could use one language construction to emulate it in plain C. This construction is logical short-circuiting.

The idea consists of:
- Separating the state we operate on in a number of objects which can besides store some information for error handling etc.
- Having operations on the state specified in terms of function that may fail (e.g. returning bool or anything which can indicate whether an arbitrary operation succeeded)

### example2.h:

```c
#include <stdio.h>
#include <stdbool.h>

struct Calculation {
    int state;
    char error[128];
};

bool operation1(struct Calculation* calculation) {
    puts(__func__);
    return true;
}

bool operation2(struct Calculation* calculation) {
    puts(__func__);
    return false;
}

bool operation3(struct Calculation* calculation) {
    puts(__func__);
    return true;
}

int main() {
    struct Calculation calculation;

    (operation1(&calculation), operation2(&calculation)) && operation3(&calculation);
    /*
     * will print
        operation1
        operation2
     */
}
```

The logical operations along with comma operator can help express complex chains of computations. In the example above operation1 is performed unconditionally and operation3 evaluation depends on the success of the operation2.
