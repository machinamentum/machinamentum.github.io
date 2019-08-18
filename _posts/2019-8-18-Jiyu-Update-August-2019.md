---
layout: post
title: Jiyu Update - August 2019
categories: [compilers, programming]
---

It has been about a month since I [last wrote about my new programming-language project, Jiyu]({% post_url 2019-7-22-Jiyu-A-Programming-Language %}), so I'm going to briefly talk about the changes that have happened since.
	
## Libraries

Libraries and macOS frameworks can now be specified using the declarators `library` and `framework`, respectively.

```swift
#if os(Windows) {
	library "msvcrt";
	library "opengl32";
}

#if os(MacOSX) {
	library "c";
	framework "OpenGL";
}
```

These inform the built-in linker command on what libraries to link, and will eventually inform the LLVM JIT on what libraries to load.

## Modules

The compiler now looks for module-libraries in the `jiyu/modules` folder. The folder is located by searching up the directory tree from the executable's path for a folder named `jiyu`, then `modules` is appended to the path. This will likely change as the compiler matures, but it currently makes working with modules easy. There is a new `#import` directive to reference a module from the scope that the directive is declared in. Since modules are intended to be referenced from many parts of a program, they exist in a scope-bubble of their own and cannot see declarations in the program's global scope. Due to this, modules must also import other modules that they depend on.

```swift
#import "Array";

func main() {
	var array: [..] uint8;
	array_add(*array, 1);
}
```

## Compiler

There is now a `Build_Options` struct for specifying compilation options. Currently, only the output executable name and target-triple can be specified. Consequently, `create_compiler_instance() -> *Compiler` now accepts a `*Builder_Options` parameter. Build options for a compiler instance can not be changed after `create_compiler_instance()` is called.

```swift
struct Build_Options {
	var executable_name: string;
	var target_triple: string;
}

// ...

#import "Compiler";

func @metaprogram main() {
	var options: Build_Options;
	options.executable_name = "my_cool_program";
	options.target_triple   = "arm-none-elf-eabi";
	
	var compiler = create_compiler_instance(*options);
	
	// ...
}
```

## LLVM

* Jiyu programs can now be debugged in your favorite debugger! 

[Jiyu Program in MSVC debugger](/assets/2019-8-18-Jiyu-Update-August-2019_MSVC_debugging.png)

* Aggregate types are now passed via an invisible reference.
* As mentioned previously, one can now specify a target triple to the compiler. LLVM will honor this, enabling cross-compilation.


## Conclusion

With these few changes, the compiler and language should be a bit more practical to use. I plan to post updates on a monthly basis, granted there's enough changes, or a big enough change, that's worth talking about.

The code for the compiler can be found at: [jiyu](https://github.com/machinamentum/jiyu).
Pull requests, feature requests, and issue reports are all welcome.
