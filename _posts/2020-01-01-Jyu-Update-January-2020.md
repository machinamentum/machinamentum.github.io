---
layout: post
title: Jiyu Update - January 2020
categories: [compilers, programming]
---

Progress has been going pretty strong this past monthly-ish period.

 * There is now a #clang_import directive. This allows importing declarations from C header files directly into Jiyu scopes. Example:
 
 ```swift
 #clang_import "#include <stdio.h>";
 
 func main() {
     printf("Hello, Moonman!\n");
 }
 ```

The string following the directive can contain arbitrary C code. This works for a number of C declaration types, including structs, unions, enums, functions, and typedefs. Do note however, that only importing bindings for declarations is supported, there is no support for compiling C function bodies, and there are no plans for Jiyu to replace a C compiler. I have been using this for awhile on Windows, and have even gotten the [nuklear imgui library working without needing to manually write bindings](https://twitter.com/machinamentum/status/1211501147808247808), but this may not work well on Mac/Linux yet if you need to import system headers.

 * There is now a mechanism to enable member-function-call syntax for arrays. I have documented this in this demo below:
 
 ```swift
 #import "LibC";
#import "Array";

// Array module defines:
// __array_add<T>(arr: *[..] T, item: T)
// __array_reset<T>(arr: *[..] T)
// __array_reserve<T>(arr: *[..] T)
// __array_resize<T>(array: *[..] T, _amount: int)
// __array_pop<T>(array: *[..] T) -> T
// __array_contains<T>(array: *[..] T, item: T) -> bool
// __array_add_if_unique<T>(array: *[..] T, item: T) -> bool

func main() {
    // When the compiler sees <array>.func(...), it will attempt to transform this sequence into
    // __array_func(<array>, ...) or __array_func(*<array>, ...).
    var arr: [..] int64;
    arr.add(1);
    arr.add(2);
    arr.add(3);
    arr.add_if_unique(3);
    arr.add_if_unique(4);

    for arr printf("Value: %lld\n", it);

    printf("Popped value: %lld\n", arr.pop());

    for arr printf("Remaining Value: %lld\n", it);

    arr.reset();
    for arr printf("Remaining Value ????: %lld\n", it); // This should not output anything!

    locally_scoped_functions();
}

func locally_scoped_functions() {
    // Since this special syntax turns into a scope-lookup,
    // we can override the defaults defined in the Array module.
    func __array_add<T>(array: *[..] T, item: T) {
        printf("Hello, Sailor!");
    }

    var arr: [..] int64;
    arr.add(1);
    for arr printf("Remaining Value ????: %lld\n", it); // should not print anything!
}

 ```
 
 * There is now a `-emit-llvm` flag to dump generated LLVM IR to a file.
 * Unions are now available, simply use the `union` keyword in place of `struct`.
 * `break` and `continue` now work for while-loops.
 * `os()` now checks against LLVM's support operating system types and the target triple of the compilation. See [Triple::getOSTypeName](https://llvm.org/doxygen/Triple_8cpp_source.html#l00175) for a list of valid inputs. The input is case-insensitive.
 * Raspberry Pi 4 is now officially supported. [Relevant tweet](https://twitter.com/machinamentum/status/1206056214028787712).
 * Diagnostic-esque information is no longer printed to the terminal by default. There is now a `-v` flag to print out this information for debugging. There's also a `verbose_diagnostics` Build_Option available to get this information out of a compilation through a metaprogram.
 * while-loop conditions now coerce-to-bool.
 * There are now built-in typealiases to reference the types for \<string\>.length and \<array\>.count fields. They are `__builtin_string_length_type` and `__builtin_array_count_type`.
 * Global variable structs are now properly default-initialized based on the default initializations of their fields.
 * [castano](https://github.com/castano) has added support for referencing let-constants declared within structs.
 * [castano](https://github.com/castano) has added a Sublime Text highlighting extension.
 * [castano](https://github.com/castano) has added support for multiline string literals similar to how they are implemented in Swift:
 
 ```swift
 
#import "Basic";
#import "LibC";

let singleLineString = "These are the same.";

let multilineString = """
These are the same.
""";

let multilineString2 = """These are the same.""";

let multilineString3 = """
    These are the same.
    """;

func main() {
    // Multi-line string equivalency.
    printf("%.*s\n", singleLineString.length, singleLineString.data);
    printf("%.*s\n", multilineString.length, multilineString.data);
    printf("%.*s\n", multilineString2.length, multilineString2.data);
    printf("%.*s\n", multilineString3.length, multilineString3.data);
    assert(singleLineString == multilineString);
    assert(singleLineString == multilineString2);
    assert(singleLineString == multilineString3);

    let indentation = """
        This text has indentation.
        And that is fine.
        """;

    printf("%.*s\n", indentation.length, indentation.data);

}
 ```
 
 * Support for passing and returning structs to/from C has been improved.
 * There are now `strideof()` and `alignof()` operators. These return the stride and alignment of any input type. Unlike C, the the size of a struct is the amount of space required to store the struct, the stride is the amount of space needed to store the struct with padding to its alignment.
 * There is now support for unary `!` and unary `~`.
 * There is now support for a rudimentary level of struct inheritance:
 
 ```swift
 func test_inheritance() {
    struct Node {
        var i = 10;

        func do_a_thing() {
            printf("Node do_a_thing()\n");
        }

        func do_a_thing2(n: *Node) {
            printf("Node.i: %d\n", n.i);
        }
    }

    struct Node_Child : Node {
        var b = "test";

        func do_a_thing3(n: *Node_Child) {
            printf("Node_Child.i: %d\n", n.i);
            printf("Node_Child.b: %.*s\n", n.b.length, n.b.data);
        }
    }

    var child: Node_Child;
    child.do_a_thing2();
    child.do_a_thing3();

    Node_Child.do_a_thing();
}
 ```

I want to again thank the contributors for picking up this project and helping to improve it!

The code for the compiler can be found at: [jiyu](https://github.com/machinamentum/jiyu).
Pull requests, feature requests, and issue reports are all welcome.
