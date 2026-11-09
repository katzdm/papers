---
title: "FI-067: Selectively transparent `$using-declarator$`s"
document: D4466
date: today
audience: SG7, EWG, CWG
author:
    - name: Dan Katz
      email: <katzdm@gmail.com>
    - name: Tomasz Kamiński
      email: <tomaszkam@gmail.com>
toc: true
status: progress
tag: reflection
---

# Background

## Sofia 2025
In the final weeks leading up to the Sofia 2025 meeting (at which the C++26 Reflection papers were accepted into the Working Draft), the ongoing Core wording review of [@P2996R13] revealed two defects:

1. The wording approach used to specify _splice-template-arguments_ was inadequate, and no viable alternative wording strategy could be identified.
2. The inability for Reflection to observe _using-declarators_ led to an incomplete treatment of access control by Reflection.

[@P3687R1] was brought to EWG to address these shortcomings. The room agreed with the authors' recommendation to remove _splice-template-arguments_ from C++26, with the intention that the idea could be brought back at a later time.

In contrast to the issues with _splice-template-arguments_ (which the P2996 co-authors had no suggestion for how to address), [@P3687R1] included a design, complete with wording, for what to do about _using-declarators_. The proposed direction was to elevate the standing of _using-declarators_, such that they not only introduce a name into a scope, but that they in fact introduce one or more entities, called _entity proxies_, for each entity named by the `$using-declarator$`. Such entity proxies could then be represented by `std::meta::info` values, allowing their properties (e.g., which entity they proxy) to be introspected in the expected ways (i.e., via metafunctions in the `std::meta` namespace).

[@P3687R1] was seen by EWG during a 2025-06-05 telecon. Given the late hour at which the suggested direction arrived, EWG declined to adopt the Entity Proxies proposal for C++26. It was instead decided that `^^foo` would be ill-formed when `foo` names a `$using-declarator$`, leaving open the possibility to reconsider Entity  Proxies for C++29. This is the status quo.

## C++26 DIS comments
In the time since Sofia, GCC 16 has shipped C++26 Reflection; this has provided an opportunity for C++ users throughout the world to gain experience with the feature. Consequently, a sharp edge was discovered when using Reflection with libstdc++'s `std` module, which exports entities from the `std` namespace in a manner like the following:

```cpp
export namespace std
{
  using std::basic_string;
  using std::char_traits;
  // ...
}
```

This makes programs like the following ill-formed:

```cpp
import std;

constexpr std::meta::info R = ^^std::basic_string;  // error: cannot take the reflection of a using-declarator.
```

even though the equivalent program using `#include` is well-formed:

```cpp
#include <meta>
#include <string>

constexpr std::meta::info R = ^^std::basic_string;  // OK
```

The Finnish national body published a comment (FI-067) against the C++26 DIS voicing their concern around this restriction:

> It appears there's various difficulties with using reflection on standard library entities exported from modules. In at least libstdc++, what gets exported is a `$using-declaration$`, and reflection of such can’t be taken. There is an ugly `^^typename std::string` work-around, but no similar work-around even exists for taking a reflection of a template exported from a module.

Entity Proxies would address this comment, but we believe that the concept need more time to bake: both more time dedicated to the specification wording, and more time for the community to experiment with and evaluate Entity Proxy implementation(s). Rather than try to shove pieces of [@P3687R1] into C++26 at the fourteenth hour, we propose a far more narrow change.

# Design

## Proposed direction
We propose a narrow carveout to make the following well-formed:

```cpp
namespace NS {
void f();
using NS::f;

}  // namespace NS

constexpr std::meta::info R = ^^NS::f
  // ok, using-declarator names declaration in its own namespace scope
```

as well as this:

```cpp
namespace NS {
inline namespace Inline {
void f();

}  // namesapce Inline

using Inline::f;
}  // namespace NS

constexpr std::meta::info R = ^^NS::f;
  // ok, using-declarator only separated from its named declaration by an inline namespace
```

as well as this:

```cpp
namespace NS {
namespace {
void f();

}  // anonymous namespace
}  // namespace NS

constexpr std::meta::info R = ^^NS::f;
  // ok, using-declarator only separated from its named declaration by an unnamed namespace
```

while this will remain ill-formed:

```cpp
namespace NS {
void f();

}  // namespace NS

using NS::f;
contstexpr std::meta::info R = ^^f;
  // error: cannot reflect using-declarator separated from its named declaration by 'namespace NS'.
```

as well as this (also still ill-formed):

```cpp
struct Base {
  void f();
};
struct Derived : Base {
  using Base::f;
};

constexpr std::meta::info R = ^^Derived::f;
  // error: cannot reflect using-declarator 'Derived::f'.
```

This preserves the syntactic space for reflection over `$using-declarator$`s for C++29 (or whatever), while fixing the painful case pointed out by FI-067.


## Design tradeoffs

Because it will become possible to write the following program:

```cpp
namespace NS {
namespace Inline {
void f();  // #1

}  // namespace Inline

using Inline::f;  // #2

}  // namespace NS

static_assert(^^NS::f == ^^NS::Inline::f);  // ok, probably forever
static_assert(std::meta::parent_of(^^NS::f) == ^^NS::Inline);  // ok, probably forever
```

The adoption of this paper would make it unlikely, even with the future adoption of an Entity Proxies paper, that the expression `^^NS::f` could ever be made to refer to `#2` instead of `#1`. This could be viewed as "weakening" the power of the reflection operator, since certain information (i.e., the `$using-declarator$`) is "erased" from its view.

On the other hand, we argue that -- for the cases allowed by this paper -- it is unlikely that a user _wants_ `^^NS::f` to refer to the `$using-declarator$`: it is far more likely that the user is trying to take the reflection of `void f()`, and the `$using-declarator$` is a mere "implementation detail" of how a library author "surfaced" `void f()` to the user. This very well may, therefore, only increase the ergonomics of the reflection operator.

It's important to note that although `^^NS::f` might never refer to the `$using-declarator$`, the `$using-declarator$` need not be forever invisible to Reflection: It still ought to be possible (and desirable) for `std::meta::members_of(^^NS, Ctx)` (for some `Ctx`) to surface a representation of the `$using-declarator$` in the resulting `std::vector<std::meta::info>`. We therefore get the best of both worlds: `^^NS::f` refers to the underlying function (which is probably what the user intends), while the `$using-declarator$`, through which that function was found during lookup for `NS::f`, remains observable via the more structured, graph-walking introspection facilitated by `std::meta::members_of`.


# Wording
Modify [expr.reflect]/5 as follows:

::: wording
[5]{.pnum} If a `$reflect-expression$` _R_ matches the form `^^$reflection-name$`, it is interpreted as such; the identifier is looked up and the representation of _R_ is determined as follows:

- [#.#]{.pnum} If lookup finds a declaration [_D_]{.add} that replaced a `$using-declarator$` [_U_]{.add} during a single search ([basic.lookup.general], [namespace.udecl]), [_R_ is ill-formed unless]{.rm} [the target scope of _U_ shall be associated with a namespace, and each scope (if any) that intervenes between the target scope of _U_ and a point whose immediate scope is the target scope of _D_ shall be associated with an inline or unnamed namespace ([namespace.def.general, namespace.unnamed])]{.add}.
- [#.#]{.pnum} [Otherwise, if]{.rm} [If]{.add} lookup finds a namespace alias ([namespace.alias]), all declarations found by name lookup shall have the same target scope, and _R_ represents the namespace alias.
- [#.#]{.pnum} Otherwise, if lookup finds a namespace ([basic.namespace]), _R_ represents that namespace.
- [#.#]{.pnum} Otherwise, [...]

:::


---
references:
---
