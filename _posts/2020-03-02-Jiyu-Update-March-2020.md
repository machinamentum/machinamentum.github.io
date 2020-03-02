---
layout: post
title: Jiyu Update - March 2020
categories: [compilers, programming, jiyu]
pitfall_demo_id: a1_7y4wQZMc
---

February has been a bit of a slow month, but we've managed to add a few interesting features.


 * Added support for default parameters:

```swift
#import "LibC";

func test(a: int32 = 10, b: float = 1, c = "hello") {
    printf("a: %d\n", a);
    printf("b: %f\n", b);

    printf("c: %.*s\n", c.length, c.data);
}

// We don't currently error on the case that two or more functions
// cause an ambigous overload situation. Currently, the behavior
// is that the functions with fewer needed default params is prefered.
// This means you cannot call the above function via test(2, 3), or test(1), as
// the function below will be chosen instead since it requires less parameters
// to be filled in.
func test(a: int32, b: float = 1) {
    printf("Dang\n");
}

func main() {
    test();
    test(1);
    test(2, 5, "test");

    // Will print "Dang"
    test(3, 5);
}
```

 * Added support for binary literals, starting with the prefix `0b` or `0B`.
 * Added support for switch-case control flow:

```swift
 #import "LibC";

func main() {

    enum Test {
        A;
        B;
        C;
        D;
    }

    var i = Test.D;

    switch i {
        case .A:
            printf(".A\n");
        case .B:
            printf(".B\n");

        case .C, .D:
            printf(".C or .D\n");
    }

    let Y = 10;
    var x = 2;

    // Currently only integer and enum (integer derived) types are support.
    switch x {

        // Currently only expressions that resolve to integer literals are supported.
        case 1:
            printf("1\n");
        // Multiple case conditions can be declared per case-block.
        case 2, 3:
            printf("2\n");
        case Y:
            printf("Y\n");
    }

    printf("Done\n");
}
 ```

 * Added support for overloading the array index operator `[]`, and the lvalue-assignment array index operator `[]=`:

```swift
operator[] <T, R>(arr: [..] T, index: R) -> T {
    assert(index >= 0 && cast(int64) index < arr.count, "Array index out of range!");
    return arr.data[index];
}

operator[]= <T, R>(arr: [..] T, index: R, value: T) {
    assert(index >= 0 && cast(int64) index < arr.count, "Array index out of range!");
    arr.data[index] = value;
}

 ```

 * Added support for template structs:

```swift
#import "LibC";

struct Poly<T> {
    var a: T;

    func do_thing(this: *Poly<T>) {
        if (T == int)   printf("%d\n", this.a);
        if (T == float) printf("%f\n", this.a);

        // Unfortunately, we cannot straight up do this because this is invalid when T == int or T == float
        // if (T == string)printf("%.*s\n", this.a.length, this.a.data);

        // But if we added another static-if construct like `when` in Odin then
        /*
            when (T == string) printf("%.*s\n", this.a.length, this.a.data);
        */
        // would work. I do not want to make #if this generous as #if operates directly on
        // the scope it is declared in, `when` on the other hand would work exactly like a
        // a regular `if` except the body is only evaluated when the condition is a literal
        // `true` (or folds to).
    }
}

func main() {
    var a: Poly<int>;
    a.a = 10;
    a.do_thing();

    var b: Poly<float>;
    b.a = 1345.234;
    b.do_thing();

    var c: Poly<string>;
    c.a = "Hello, Pilot!";
    c.do_thing();

    printf("%.*s\n", c.a.length, c.a.data);

    // We currently do not have a way to parse TypeName<Typearg, Typearg2, etc>
    // in regular expressions, so currently one cannot use templates in first-class
    // uses unless one uses a typealias (since the right hand side of a type alias is
    // always a type instantiation).
}
```

 * Added an intrinsic function for inserting a debugging breakpoint in code `__builtin_debugtrap()`.
 * [castano](https://github.com/castano) added support for filling function types in polymorphic parameters:

 ```swift
#import "Basic";
#import "LibC";

func test<T> (a: T, b: T, comp: (a: T, b: T) -> int) -> int {
    return compare(a, b);
}

func compare(a: int, b: int) -> int {
    return a - b;
}

func bar<T> (foo: () -> T) -> T {
    return foo();
}

func zero () -> int {
    return 0;
}

func main () {
    var i = bar(zero);
    var j = 1;

    var result = test(i, j, compare);
    assert(result == -1);

    printf("result = %d\n", result);
}


 ```

 * [castano](https://github.com/castano) added support for declaring float literals in scientific notation.
 * Added support for anonymous unions and anonymous structs:

```swift
#import "LibC";

struct Test {
    var d: float;

    union {
        var a: int;
        var b: int;
    }

    var c: int;
}

func main() {
    var t: Test;

    t.a = 10;
    t.c = 20;
    t.d = 15.5;

    printf("t.d: %f\n", t.d);
    printf("t.a: %d\n", t.a);
    printf("t.b: %d\n", t.b);
    printf("t.c: %d\n", t.c);
}
```

 * Upgraded to LLVM 9. This change has been made primarily to pull in newer features from libclang inorder to properly detect and import anonymous unions/structs.

Additional changes include various fixes.

## Projects

One of the main reasons I am building this language is to have fun programming in the language myself. In early February, I had decided to start a new project that could be built within a reasonably short amount of time, but still do something that is interesting and relatively nontrivial. I chose to build an Atari 2600 emulator since I have had previous experience building this type of emulator, and it fits within this criteria: the 6507/6502 has a relatively small instruction set and is well documented, the 2600 has a fairly small number of hardware interface registers, and thus has a small surface area to implement for each of the onboard chips, and the hardware's behavior is fairly simple. That being said, the hardware does have a large number of minor quirks, but if you are not aimaing for 100% accuracy, then a mostly naive implementation of the hardware from the perspective of the Stella programmer's manual is enough to get a good number of games working in a relatively small amount of time.

I am planning to do a separate write up on the project in the near future. The source code can be found [here](https://github.com/machinamentum/HM2600). I've uploaded a Windows binary [here](https://github.com/machinamentum/HM2600/releases/tag/release_1).

{% include youtubePlayer.html id=page.pitfall_demo_id %}

## Conclusion

I want to again thank the contributors for picking up this project and helping to improve it!

The code for the compiler can be found at: [jiyu](https://github.com/machinamentum/jiyu).
Pull requests, feature requests, and issue reports are all welcome.
