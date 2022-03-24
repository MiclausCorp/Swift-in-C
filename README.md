## Welcome
This guide will teach you how to call a Swift function from C/C++, and pass data between them.

## Writing the Swift code

First, create a Swift Source file named **main.swift** and copy the following code:
```swift
@_cdecl("greet")
public func greet() {
   print("Hello, World!")
}
```
This is a regular "Hello, World!"-example wrapped in a function called `greet()`.

The only thing that stands out, is the Function Attribute `@_cdecl("greet")`.

### But what is `@_cdecl` ?
cdecl (which stands for C declaration), is an **internal** and **undocumented** function attribute in the Swift Compiler, which according to vague information found on the web, enforces C-name mangling. 

Note: `@_cdecl` solely enforces the name mangling; not the calling convention!

## Creating a Dynamic Linked Library
In order to link our code with a C/C++ application, will need a library. 

In order to compile our code as a linked library, **specify `-emit-library` when compiling the .swift file.**
```shell
swiftc main.swift -emit-library -o libGreet.dylib
```

This will result in a file library called "libGreet.dylib" (or "libGreet.so" on Linux). If you'd open a Hex viewer right now and examine the file, you'll see it exported the function symbol as "_greet".

## Writing the C/C++ Code
We'll use C++ for the purposes of this guide. Create a C++ Source file named **main.cpp** and copy the following code:
```cpp
// only "extern" when targeting C++.
extern "C" void greet();

int main() {
    greet();
    return 0;
}
```
Here we declare `greet()` as an external reference, which will be provided by the library libGreet.dylib, we compiled before.

## Compiling and Linking
What's left at this point is just to compile and link our code. 

We're using C++, so we'll call clang++, pass it our libGreet.dylib library, the main Source file and specify an output file.
```
clang++ libGreet.dylib main.cpp -o main
```

When you run the application, you'll be greeted with following:
```
./main
> Hello, World!
```
Congratulations, you've just called Swift code from a library, in C/C++

## Passing Arguments
`@_cdecl` directive also forces the cdecl-calling convention, which trivially enables us to pass arguments.

Create a Swift Source file named **Square.swift** and copy the following code:
```swift
@_cdecl("square")
public func square(_ input: Int32) -> Int32 {
   return input*input
}
```
This is a function which takes in an argument `input` and returns an 32-bit integer.

Now compile the Swift source code file to a Dynamically Linked Library using:
```
swiftc Square.swift -emit-library -o libSquare.dylib
```

Now modify **main.cpp** to the following:
```cpp
#include <cstdint>
#include <iostream>

extern "C" std::int32_t square(std::int32_t);

int main() {
    std::cout << square(2) << std::endl;
    return 0;
}
```
Here we're passing *2* to the `square(_)` Swift function, and we expect to get something back, so we're also printing it to the Standard Output.

Now compile and link the C++ source code file using:
```
clang++ libSquare.dylib main.cpp -o main
```

When you run the application now, you'll see the following:
```
./main
> 4
```
You'll see that it prints *4*, which is indeed, the square of *2*.

You're free to mess around with this and see what you can do with it. But don't expect future stability and definetly don't use this in a production application. `@_cdecl` is afterall, an internal function attribute.

You may also be interested in this:
[The Swift and C++ interoperability workgroup](https://forums.swift.org/t/swift-and-c-interoperability-workgroup-announcement/54998)
