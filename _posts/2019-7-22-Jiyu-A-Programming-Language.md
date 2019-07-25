---
layout: post
title: Jiyu - A Programming Language
categories: [compilers, programming]
---

Every so often, I am introduced to a new programming language or compiler project that, at the surface level, seems interesting, but leaves a lot, for me, to be desired. Several of these include relatively new, currently-used-in-production languages, such as Swift, where ideological constraints on the language design have made it difficult to write general purpose programs that exist outside the scope of the language’s primary, intended use. Additionally, I have some reservations about the language and compiler I work on professionally. I still yearn for something to meet my needs as a programmer. I still need to write programs that manually manage memory; I still need to write code that runs at CPU boot-time; I still need a lot of C++’s convenience.

Back in 2015, I had started a compiler project, called [htn](https://github.com/machinamentum/htn), with just the goal of being able to compile code, interact with C-code, and output to a handful of assembly-language targets, including 6502, 32-bit ARM, and 32-bit x86. I’ve since dropped the project due to other projects and my day-job at-the-time consuming much more of my free-time. I had been thinking about this project often over the years as I had enjoyed the work I was doing on it. Unfortunately, the code isn’t maintainable for me anymore, I have grown considerably as a programmer since and am unwilling to change substantial portions of the code just to get it to a point of maintainability. Doing so would require nearly starting over again; so I started over, again.

## Jiyu

This time around, I’ve built an LLVM-based compiler with an out-of-order compilation model not unlike any modern language created in the last 5 or so years. My initial goals are quite simple: implement an appealing syntax that reduces friction and enables ease of refactoring, have the ability to just drop the compiler and its dependencies into existing toolchains for cross-compiling, and be able to manipulate memory unencumbered. Fortunately for me, the former and latter-most issues have largely been solved by other projects in this space. I really, _really_, enjoy Swift’s syntax, but I despise its type system as the design of it makes writing code, that needs to read and write unmanaged memory, difficult. I also really like Jai’s type system; it is reminiscent of C’s, but with built-in array, and string, types, and less confusion. I’ve opted to adopt both these things, let’s look at some code:

```swift
func @c_function printf(fmt: *uint8, temporary_c_vararg) -> int;

func main() {
	var b = cool_function(2);
	printf(“Value of b: %f\n”.data, b);
}

func cool_function(a: float) -> float {
	return a * 2;
}
```

Most of the syntax here should be familiar to anyone who’s done even a light amount of Swift programming, with a few notable exceptions. The tag, `@c_function`, informs the compiler that we’re declaring a function that is external C-code, and to take care in calling this function correctly. `temporary_c_vararg` is fancy placeholder for the `...` syntax for a C-variable-argument function. Since strings are represented by a small struct containing the length of the string and a point to its data, we must dereference the data pointer before we can pass it to `printf`.

## User-types

For user-types, the language currently supports type-aliasing and structs:

```swift
typealias c_string = *uint8;

func @c_function printf(fmt: c_string, temporary_c_vararg);

typealias FILE = void;
func @c_function fopen(path: c_string, mode: c_string) -> *FILE;
func @c_function fclose(file: *FILE) -> int32;

struct Matrix {
	var data: [4][4] float;
}

func identity_matrix() -> Matrix {
	var m: Matrix;
	
	m.data[0][0] = 1;
	m.data[1][1] = 1;
	m.data[2][2] = 1;
	m.data[3][3] = 1;
	
	return m
}
```
Variables are zero-initialized by default in this language. There’s currently no way to declare an uninitialized variable, but that is on the list of things to do.
Structs may be populated by any other type of declaration, including functions. There’s currently no notion of member-functions, however, that’s another thing to come in the future.

```swift
struct Matrix {
	var data: [4][4] float;
	
	let SIZE = 16;
	
	func ortho(left: float, right: float, top: float, bottom: float, near: float, far: float) -> Matrix {
		// ...
	}
	
	func identity() -> Matrix {
		// ...
	}
}

func main() {
	var matrix = Matrix.identity();
	// ...
	var size = Matrix.SIZE;
}
```

## Static-if
Another borrowed feature from Jai, is a scope-aware static-if:

```swift
func main() {
	#if os(MacOSX) {
		var b = 1;
	}
	#if os(Windows) {
		var b = 2;
	}
	b = 3;
}
```

The usefulness of this is that whenever code in a disabled static-if block is changed, it is still being parsed, enabling syntax-specific errors to still fire. `os()` is a special construct that checks the input for a known operating-system name, then resolves to a Boolean literal if that operating system happens to be the target platform. This is not too dissimilar to how `sizeof()` works, except, in this case, “MacOSX” and “Windows” are special identifiers that `os()` recognizes. Due to the complexity of allowing arbitrary expressions in the `#if` condition, I’ve specifically disallowed arbitrary expressions. Only expressions that specifically fold or resolves to literal values may appear here; otherwise, even just allowing `let` constants in the condition causes a number of dependency issues relating to when that `let` exists in a scope that the `#if` can see. We cannot have `#if` depend on information that only exists after another `#if` or `#load` has resolved in order to maintain simplicity in the out-of-order compilation system.

## Templates

There’s currently limited support for function-templates. Struct-templates will will be on their way, eventually. Template declaration and resolution rules are similar to Swift’s: template argument names are declared between `<` and `>`, the type of the template arguments are deduced by arguments passed to the template function.

```swift
let NEW_MEM_CHUNK_ELEMENT_COUNT = 16;

func array_reserve<T>(array: *[..] T, _amount: int) {
    var amount = _amount;
    if amount <= 0
        amount = NEW_MEM_CHUNK_ELEMENT_COUNT;

    var new_mem = cast(*T) malloc(amount * sizeof(T));

    if array.data != null {
        memcpy(new_mem, array.data, cast(size_t) (array.count * sizeof(T)) );
        free(array.data);
    }

    array.data = new_mem;
    array.allocated = amount;
}

func array_resize<T>(array: *[..] T, _amount: int) {
    array_reserve(array, _amount);
    array.count = _amount;
}

func array_add<T>(array: *[..] T, item: T) {
    if (array.count+1 >= array.allocated) array_reserve(array, cast(int) 
    
    array.allocated + NEW_MEM_CHUNK_ELEMENT_COUNT);

    array.count = array.count + 1;
    (<<array)[array.count-1] = item;
}

func array_reset<T>(array: *[..] T) {
    array.count = 0;
    array.allocated = 0;

    if array.data != null free(array.data);
    array.data = null;
}

func main() {
	var data: [..] float;
	
	array_add(*data, 1.0);
	array_add(*data, 2.0);
}
```

## Compile-Time Function Execution

There’s rudimentary support for CTFE. One may run any program at compile-time by invoking the `-meta` flag on the command line: ```jiyu -meta build.jyu```. Additionally, the `main` function may be marked by the tag `@metaprogram` to force the compiler to always run that program at compile-time. Just a small change to our first code example will cause the program to always be executed at compile-time:

```swift
func @c_function printf(fmt: *uint8, temporary_c_vararg) -> int;

func @metaprogram main() {
	var b = cool_function(2);
	printf(“Value of b: %f\n”.data, b);
}

func cool_function(a: float) -> float {
	return a * 2;
}
```

Compile-time code may also harness the compiler’s main driver functions to compile, or run, other code. I won’t delve too deeply into that, but here is a code sample:

```swift
#load "Compiler.jyu";

func @metaprogram main() {
    var compiler = create_compiler_instance();
    compiler.executable_name = "run_tree/game";
    
    if compiler_load_file(compiler, "src/main.jyu") != true return;
    if compiler_typecheck_program(compiler) != true return;
    if compiler_generate_llvm_module(compiler) != true return;
    if compiler_emit_object_file(compiler) != true return;
    if compiler_run_default_link_command(compiler) != true return;
}
```

## Conclusion

This has been a brief look at the language. There are still a number of small issues to tackle (like specifying libraries the code depended on; currently the compiler has the linker command baked-in), but I plan to fix these within the coming weeks. Please see the [jiyu_game](https://github.com/machinamentum/jiyu_game) project for more in-depth use of the language.

The code for the compiler can be found at: [jiyu](https://github.com/machinamentum/jiyu).
Pull requests, feature requests, and issue reports are all welcome.
