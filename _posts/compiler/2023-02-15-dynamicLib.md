---
title: Dynamic Library
author: Tao Sun
date: 2023-02-15
category: compiler
layout: post
---

This post is posted on [creating and using dll](https://gernotklingler.com/blog/creating-using-shared-libraries-different-compilers-different-operating-systems/)

```
"A shared library or shared object is a file that is shared by executable files and further shared objects files.” A shared library on Linux is called “dynamically linked shared object”, and has the file extension .so. The windows equivalent is the “dynamic link library” usually with file extension .dll."
```

# Linux/GCC
```
.
├── shared.h
└── shared.cpp
```

Let’s start building our demo on Linux with GCC. The header of our shared library is shared.h and its implementation is in shared.cpp. Our application is implemented in main.cpp, where we want to use our shared library.

shared.h:
```cpp
#ifndef SHARED_H__
#define SHARED_H__

void f();

class X {
public:
  X();
  void mX();
};

#endif
```

shared.cpp:
```cpp
#include "shared.h"
#include <iostream>

void f() {
  std::cout << "f()\n";
}

X::X() {
  std::cout << "X::X()\n";
}

void X::mX() {
  std::cout << "X::mX()\n";
}
```
**Now let’s compile our library with GCC:**

```
g++ -fPIC -shared shared.cpp -o libshared.so
```
ls result
```
.
├── shared.h
├── shared.cpp
└── libshared.so
```

main.cpp:
```cpp
#include "shared.h"

int main() {
  f();
  X x;
  x.mX();
}
```
```
g++ -L. -lshared -o main main.cpp
```

This will raise error: 
```bat
./main: error while loading shared libraries: libshared.so: cannot open shared object file: No such file or directory
```
To solve this problem, we must somehow specify where the library can be found. On Linux, the environment variable LD_LIBRARY_PATH can be used to specify directories where libraries are searched for first (before the standard set of directories)((GNU Dynamic Loader search directories)) => prepending our commands above with LD_LIBRARY_PATH=./ solves the issue of the not found library and the program will execute like expected:
```
$> LD_LIBRARY_PATH=. ./main
f()
X::X()
X::mX()
```

# MinGW
- <font color=red>Symbol Visibility: GCC/MinGW exports all symbols by default, MSVC exports no symbol by default.</font>

if
- u are on windows
- compile everything with MinGW 

Then the code same as above works.
```
g++ -fPIC --shared -o shared.dll shared.cpp
g++ -L. -lshared -o main main.cpp
```
Like shown above, it is perfectly fine to 
directly link the DLL if the DLL was also created by MinGW. But this is not the common way to work with shared libraries on windows because just a few compilers support this - the Microsoft Visual Studio Compiler is not among them.

The default behavior of MinGW is to export ALL symbols of a DLL. But if one or more functions are declared with, only those functions are exported.

# MSVC
- <font color=red> MSVC cannot directly link to a dll -> we need a so-called “import library”</font> 

The behavior of the Microsoft Visual Studio Compiler is different: Per default NO symbols are exported, just symbols explicitly declared with __declspec(dllexport).

So let’s make the code “MSVC compatible”, without loosing the compatibility to the other compilers:

We add an additional file shared_EXPORTS.h:
```
#ifndef _SHARED_EXPORTS_H__
#define _SHARED_EXPORTS_H__

#ifdef _WIN32
    #ifdef shared_EXPORTS
        #define SHARED_EXPORT __declspec(dllexport)
    #else
        #define SHARED_EXPORT __declspec(dllimport)
    #endif
#else
    #define SHARED_EXPORT
#endif

#endif /* _SHARED_EXPORTS_H__ */
```
and modify our shared.h as follows:
```cpp
#ifndef SHARED_H__
#define SHARED_H__

#include "shared_EXPORTS.h"

void SHARED_EXPORT f();

class SHARED_EXPORT X {
public:
  X();
  void mX();
};

#endif
```
We compile the code now similar like before but we define shared_EXPORTS by passing -Dshared_EXPORTS to gcc when compiling shared.cpp:

```
g++ -Dshared_EXPORTS -c shared.cpp
g++ -shared -o shared.dll shared.o -Wl,--out-implib,libshared_dll.lib
g++ -o main.exe main.o -L. -llibshared_dll
```
When using the library (i.e. linking it to main.exe), shared_EXPORTS is not defined, and SHARED_EXPORT is set to __declspec(dllimport). This means when we build the library, our functions are exported and when we want to use our library, the functions are imported from the dll.


