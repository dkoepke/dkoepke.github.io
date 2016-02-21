---
title: "Ordering of cout/cerr on coderpad.io"
tags: [coderpad.io, interviewing, software engineering]
category: etc
date: 2016-02-15
---

[Coderpad.io](https://coderpad.io) is a great tool for doing technical phone screens. However, the lack of a debugger does mean that when a interviewee's code isn't working as expected, you must resort to "output-based debugging". (On the plus side, it does print stack traces for seg faults.) One caveat with this is that Coderpad captures `cerr` and `cout` separately, then outputs `cout` messages before `cerr`. This causes the order of the output to not match the order of the execution of the statements. For example:

```cpp
#include <iostream>
using namespace std;

int main()
{
    cout << "One" << endl;
    cerr << "Two" << endl;
    cout << "Three" << endl;
    cerr << "Four" << endl;
}
```

will print:

```
One
Three
Two
Four
```
