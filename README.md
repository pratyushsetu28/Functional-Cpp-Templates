# CppLibrary

A small library of my C++ stuff built upon my header library.
The header library is meant to work similarly to the type meta programming part of Boost.Hana while alleviating some issues I had with it.
The approach is a functional one, relying on template programming instead of constexpr object representation. It maps untyped lambda calculus with strict evaluation to structures available in C++-1z.

The code quality is not up to Boost.Hana yet but afterall I explore a completely new style of functional templating, sorry for the inconvenience. A very interesting aspect of the Boost library is the usefullness of its error messages which I can currently not even come close to. Debugging statements are rare (only tested for correctness by a couple of test cases) and "type type safety" is not enforced in any way.

Paradigm
-------
The only first class citizen in headerlib are structs. Without free template parameters of any sort. It's is very complicated for functions in the template system to have side effects on the type system which is strong reasoning behind the functional approach of programming I chose for this library.

Several conversion methods from conventional template meta programming approaches to the one supported here exist, for easy of use. This is by design, the user should be able to incorporate his own style and methods of MTP while being offered the full depth and power the library has to offer.

```
// Transforming a template from std in one line
using add_ptr = TypeFunction<::std::add_pointer>;
```

Building
--------
Make sure to have clang-4.0 installed on your system and at least gcc-libs version 6 (I may try to remove all std dependencies of core in future releases). Then you can simply run
```scons
```

Otherwise, any compilter compatible to c++ draft n4659 should work, you can specify it as follows:
```CXX=<compiler> scons
```
Note that gcc-7.1 is not compatible at the moment.

Advantages
----------
Let's look at an example as a comparison from Boost.Hana about eval_if coding the factorial function [adapted for c++17 from here](http://www.boost.org/doc/libs/1_62_0/libs/hana/doc/html/group__group-Logical.html#gab64636f84de983575aac0208f5fa840c)
```template <typename N>
template<typename N>
constexpr auto ffactorial(const N n) {
    return hana::eval_if(hana::equal(n, hana::int_c<0>),
        [] () constexpr { return hana::int_c<1>; },
        [=](auto _) constexpr { return hana::mult(n, fact(hana::minus(_(n), hana::int_c<1>))); }
    );
}
constexpr auto res = ffactorial(hana::int_c<5>);
```
There are several negative points made in the documentation and it's just as bad when using it to dynamically determining a typename.
But you know how to code the same function with template specialization already. That's where this library comes in.
```
template<int n> struct factorial {
  constexpr static const int value = n*factorial<n-1>::value;
};
template<> struct factorial<0> {
  constexpr static const int value = 1;
};
constexpr auto res = hdr::Apply<hdr::ValueTemplateFunction<factorial>, hdr::Signed<5>>::value;
```
To see this in action, see [this file](test/src/math.cpp)


Real world application
---------
Consider building a parser program for a specific language. Several approaches to this problem have already been explored (e.g. [YACC][YACC], [CUP][CUP]) but most suffer from a minor flaw:
They generate the source files instead of being integrated into them, or their products suffer from the additional overhead of being generated at program runtime.

This can theoretically be tackled with this library. Take an existing parser generator in a functional language, such as [Parsec][PARSEC] and reformulate the well revised code for this library. Then one is able to make use of every standard conformant c++ compiler to generate a parser, fully contained within the source file. With a guarantee of being runtime-overhead free compared to a version generated by an external tooling with the same parser structure.

Should reflection (even as simple as [N4428][N4428]) be integrated into C++ one day, these two tools could fuse into a truly powerful machinery capable of fully replacing Qt-Moc in a way that every programmer can read and understand when he has knowledge about C++.

[YACC]: http://dinosaur.compilertools.net/yacc/
[CUP]: http://www2.cs.tum.edu/projects/cup/
[PARSEC]: https://hackage.haskell.org/package/parsec
[N4428]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4428.pdf
