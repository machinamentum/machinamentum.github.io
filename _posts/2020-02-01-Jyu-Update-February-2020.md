---
layout: post
title: Jiyu Update - February 2020
categories: [compilers, programming]
---

Progress has been a bit slow for the last few weeks because I've personally lost a large amount of time to playing Stardew Valley, but we've managed to still fit in a number of important features for this month's update.

 * There is now support for harnessing LLVM's JIT engine in user code

 ```swift
#import "Compiler";

// Since metaprograms are just fully JIT-ed LLVM modules,
// we have been internally using the JIT-er to lookup the "main" symbol
// in a program and call that... We no longer do that internally.
// There is a legacy function in Compiler to mimick the way this worked
// previously, but now we allow user code to fully leverage this functionality.

func @metaprogram main() {
    var options: Build_Options;
    options.only_want_obj_file = true;
    var compiler = create_compiler_instance(*options);

    if compiler_load_string(compiler, CODE_TO_COMPILE) != true return;
    if compiler_typecheck_program(compiler) != true return;
    if compiler_generate_llvm_module(compiler) != true return;

    // New function to JIT the entire program.
    if compiler_jit_program(compiler) != true return;

    // Once compiler_jit_program passes, we can now arbitrarily query symbols
    // and call into them.
    var do_a_thing = cast(() -> void) compiler_jit_lookup_symbol(compiler, "do_a_thing");
    do_a_thing();

    var do_a_thing2 = cast((p: string) -> void) compiler_jit_lookup_symbol(compiler, "do_a_thing2");
    do_a_thing2("Howdy :)");
}

let CODE_TO_COMPILE =
"""
#import "LibC";

// We are using the @export tag here for the sake of demonstration, otherwise
// we would need to lookup the proper jiyu-mangled names.
func @export("do_a_thing") do_a_thing() {
    printf("Hello, Sailor\n");
}

func @export("do_a_thing2") do_a_thing2(str: string) {
    printf("Hello, Pilot! %.*s\n", str.length, str.data);
}
""";
 ```

Additionally, we now provide a library version of the compiler that is built alongside the compiler driver program. The API is exactly the same whether the user code is running as a metaprogram or as a compiled binary. This allows users to effectively implement custom driver frontends that support arbitrary operations that the official driver does not. This also allows compiled user code to JIT-compile Jiyu code at runtime, enabling games to embed Jiyu as a robust scripting system, or enabling IDEs to embed Jiyu in environments where running external executables and toolchains is not allowed.

 * Support for operator overloading has been added. Currently, only `+`, `-`, `*`, and `/` can be overloaded, but support for more operators will be added in the future.

```swift
#import "LibC";

struct Vector3 {
    var x: float;
    var y: float;
    var z: float;

    func make(x: float, y: float, z: float) -> Vector3 {
        var v: Vector3;
        v.x = x;
        v.y = y;
        v.z = z;
        return v;
    }
}

operator+(b: float, str: string) -> float {
    return b + cast(float) str.length;
}

operator+(a: Vector3, b: Vector3) -> Vector3 {
    return Vector3.make(a.x+b.x, a.y+b.y, a.z+b.z);
}

operator-(a: Vector3, b: Vector3) -> Vector3 {
    return Vector3.make(a.x-b.x, a.y-b.y, a.z-b.z);
}

func print_vector(v: Vector3) {
    printf("vector3: %f, %f, %f\n", v.x, v.y, v.z);
}

func @metaprogram main() {
    var a = Vector3.make(1, 2, 3);
    var b = Vector3.make(4, 5, 6);

    var c = a + b;
    var d = b - a;

    print_vector(a);
    print_vector(b);
    print_vector(c);
    print_vector(d);

    printf("1.0 + \"Hello\": %f\n", 1.0 + "Hello");
}

```

 * [castano](https://github.com/castano) has implemented support for `typeof()` operator that resolves to the type of an input declaration.

 * [castano](https://github.com/castano) has implemented support for enums:

```swift
#import "LibC";
#import "Basic";


enum Day : uint {
    Monday;
    Tuesday = Monday + 1;
    Wednesday = 2;

    Thursday;
    Friday = 1 + Thursday;

    Saturday;
    Sunday;
}

enum WeekendDay {
    Saturday = cast(WeekendDay)Day.Saturday;   // This should require a cast. Do not allow implicit cast from one enum to another.
    Sunday;
}

enum Bool : uint8 { False; True; }

typealias DAY = Day;

var another_day : DAY = Day.Monday;

var monday = Day.Monday;
var tuesday : Day = Day.Tuesday;
var sunday : Day = cast(Day)WeekendDay.Sunday;
var my_day : Day;

func test(day : Day) -> int {
    return cast(int)day + 1;
}

func main() {
    {
        var day : Day;

        let Monday = Day.Monday;
        let Tuesday = Monday + 1;
        assert(typeof(Monday) == Day);

        let MONDAY = cast(int)Day.Monday;   // Type of MONDAY is int
        assert(typeof(MONDAY) == int);
    }
  
    printf("Monday = %d\n", Day.Monday);
    printf("Tuesday = %d\n", Day.Tuesday);
    printf("Wednesday = %d\n", Day.Wednesday);
    printf("Thursday = %d\n", Day.Thursday);
    printf("Friday = %d\n", Day.Friday);
    printf("Saturday = %d\n", Day.Saturday);
    printf("Sunday = %d\n", Day.Sunday);

    printf("test(Day.Monday) = %d\n", test(Day.Monday));

    // Implicit calls from integer to enum:
    // Currently these are only allowed when the expression is a mutable literal.
    {
        var day : Day = 0;
        printf("test(0) = %d\n", test(0));
    }
    

    //printf("Whatever = %d\n", Day.Whatever); // This should trigger an error.
    {
        var day : Day = .Monday;
        test(.Tuesday);
        if day == .Wednesday {
            day = .Sunday;
        }
        if .Wednesday > day {
            // test(1 + .Tuesday); // This produces an error, as expected.        
        }
        
    }
    
}

```

Additionally, support for enum bit-flags is also available:

```swift
#import "LibC";
#import "Basic";

enum @flags Entity_Flags {
    Invisible;
    Solid;
    Cast_Shadows;
}

func main () {
    printf("%d\n", Entity_Flags.Invisible | .Solid | .Cast_Shadows);
}
```

 * Tuples are now part of the language:

 ```swift
#import "LibC";

func test() -> (a: int, b: float) {
    // arguments in tuple-expressions are automatically inferred
    // according to the fields of the tuple being assigned, returned, or passed to.
    return (1, 4);
}

func test3() -> (c: int, d: float) {
    return (1, 2);
}

func test2(a: (a: int, b: float)) {
    printf("T2: %d\n", a.a);
    printf("T2: %f\n", a.b);
}

func main() {
    var t = test();
    var t3 = test3();

    // t and t3 are the same type as far as the type system is concerned,
    // their tuples will share the same type information in the RTTI system,
    // and internally, they will have the same type table index,
    // however, since the semantic analysis system assigns types in-place,
    // t and t3 have distinct fields.

    // t.c = t3.a; // this fails

    printf("%d\n", t.a);
    printf("%f\n", t.b);

    printf("%d\n", t3.c);
    printf("%f\n", t3.d);

    t = t3;

    t = (2, 4);

    printf("%d\n", t.a);
    printf("%f\n", t.b);

    test2(t);
    test2((6, 9));

    var n: int;
    var m: float;

    // In the event that a tuple-expression is on the left-hand-side
    // of an assignment, the type of the tuple is directly determined by
    // the types of its arguments, regardless of the type of the tuple
    // on the right-hand-side.
    // Each field of the tuple on the right-hand-side is unpacked into
    // the individual arguments of the tuple-expression on the left-hand-
    // side.
    (n, m) = test();

    // Like other types of assignments, you cannot try to use a literal
    // in a tuple expression on the left-hand-side of an assignment.
    // (n, 1) = test(); // does not work.

    printf("n: %d\n", n);
    printf("m: %f\n", m);

    // These do not work yet, but are planned to be available eventually.
    /*
    var (i, j) = test();
    var (i: int, j: float) = test();
    */
}
 ```

Additional changes include improved support for C interfaces on various platforms, improved support for trivial-constant-folding, and various fixes.

I want to again thank the contributors for picking up this project and helping to improve it!

The code for the compiler can be found at: [jiyu](https://github.com/machinamentum/jiyu).
Pull requests, feature requests, and issue reports are all welcome.
