---
layout: post
title: Jiyu Update - November 2019
categories: [compilers, programming]
---

It has been a awhile seen I've written one of these. I had taken some time off from working on the compiler to focus my free time on learning some traditional arts. That being said, there's a number of changes to talk about so let's get into it.

 * There is now a psuedo-test-suite program located at \<jiyu>/tests.jyu. This automatically compiles the test programs in \<jiyu>/tests. This is a way to quickly verify that changes to the compiler don't break some basic features.
 * String-literals can be implicitly cast to *uint8 and *int8. This is a convinience for passing string-literals to C functions.
 * Added compiler_run_metaprogram(). This function allows metaprograms to compile and run other programs at compile-time.
 * There's now support for loading libraries that the user has specified with the _library_ keyword into metaprograms. This is a seemless operation, so there is no extra work required of the user. This may not entirely work with user-installed libraries on Linux/Mac at the moment, but this will be improved over time.
 * The compiler now accepts the "-c" switch to end compilation at generating an object file.
 * The compiler now accepts the "-o" switch to specify the target file/executable name to output to.
 * The switch "-triple" is now "-target".
 * [ahmadrosid](https://github.com/ahmadrosid) has added some missing functionality on Linux and the compiler should work on Linux now.
 * Control-flow statements _break_ and _continue_ are now available and working.
 * There is now a version of _for_ that gives a pointer to array elements instead of a const-copy of the elements. The syntax is "for * \<array_typed_expression> { ... }".
 * [castano](https://github.com/castano/) has added an open-half version of _for_, which excludes the value of the upper-bound expression of the _for_ range. Example: "for 0 ..< 2 { ... }", would be equivalent of "for { int it = 0; it < 2; it += 1) { ... }" in C.
 * _if_-conditions now have coerce-to-bool. Integer, float, and pointer-types are now automatically compared to literal-0, string is automatically compared to empty-string.
 * There is now syntax for calling member-functions of structs with more ease:

```swift
struct My_Struct {
	var i: int;
	// You can overload between a reference and a pointer.
	func do_something(data: My_Struct) { .... }
	func do_something(data: *My_Struct) { ... }
	
	func do_something2(data: *My_Struct) { ... }
	
}

func my_func() {
	var data: My_Struct;
	data.do_something(); // _data_ is implicitly passed to My_Struct.do_something(data: My_Struct);
	var data_ptr = *data;
	data_ptr.do_something(); // _data_ptr_ is implicitly passed to My_Struct.do_something(data: *My_Struct);
	data.do_something2(); // A pointer is taken of _data_ and passed to My_Struct.do_something2();
	
	// You can still call these functions the traditional way, too
	My_Struct.do_something(data);
	My_Struct.do_something(*data);
}
```


* [castano](https://github.com/castano/) has added compiler_load_string(), exposing previously internal functionality to directly compile code from strings, in metaprograms.
* Cross-compilation support has been improved. I have demonstrated [running Jiyu code on an iOS device](https://twitter.com/machinamentum/status/1195569426105536512?s=20), and [running Jiyu code bare-metal under Qemu](https://twitter.com/machinamentum/status/1193671625771769856?s=20).

Additionally, there have been a number of fixes and changes to improve the compiler, by myself and others.

There's currently work being done to import C-headers directly into Jiyu code without any external tools. I will write another post documenting that once it is working well.

I want to again thank the contributors for picking up this project and helping to improve it!

The code for the compiler can be found at: [jiyu](https://github.com/machinamentum/jiyu).
Pull requests, feature requests, and issue reports are all welcome.