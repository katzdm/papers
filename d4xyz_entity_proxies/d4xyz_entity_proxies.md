---
title: "FI-067: Entity Proxies"
document: D4xyz
date: today
audience: EWG
author:
    - name: Dan Katz
      email: <katzdm@gmail.com>
toc: true
status: progress
tag: templates
---

# Background

## Sofia 2025
In the final weeks leading up to the Sofia 2025 meeting (at which the C++26 Reflection papers were accepted into the Working Draft), the ongoing Core wording review of [@P2996R13] revealed two defects:

1. The wording approach used to specify _splice-template-arguments_ was inadequate, and no alternative wording strategy (short of a large refactor of how templates are specified) could be identified.
2. The inability for Reflection to observe _using-declarators_ led to an incomplete treatment of access control by Reflection.

[@P3687R1] was brought to EWG to address these shortcomings. The room agreed with the authors' recommendation to remove _splice-template-arguments_ from C++26, with the intention that the idea could be brought back at a later time.

In contrast to the issues with _splice-template-arguments_ (which the P2996 co-authors had no suggestion for how to address), [@P3687R1] included a design, complete with wording, for what to do about _using-declarators_. The proposed direction was to elevate the standing of _using-declarators_, such that they not only introduce a name into a scope, but that they in fact introduce one or more entities, called _entity proxies_, for each entity named by the `$using-declarator$`. Such entity proxies could then be represented by `std::meta::info` values, allowing their properties (e.g., which entity they proxy) to be introspected in the expected ways (i.e., via metafunctions in the `std::meta` namespace).

[@P3687R1] was seen by EWG during a 2025-06-05 telecon. Given the late hour at which the suggested direction arrived, EWG felt that there was not enough time to review the proposed direction. It was instead decided to make ``^^foo` ill-formed when `foo` names a `$using-declarator$`, which left open the possibility to reconsider Entity  Proxies for C++29. This is the status quo.

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

As a result of the aforementioned direction adopted by EWG in response to [@P3687R1], programs like the following become ill-formed:

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

> It appears there's various difficulties with using reflection on standard library entities exported from modules. In at least libstdc++, what gets exported is a _using-declaration_, and reflection of such can’t be taken. There is an ugly ^^typename std::string work-around, but no similar work-around even exists for taking a reflection of a template exported from a module.

Members of the Finnish national body have indicated that the Entity Proxies direction, described and specified by [@P3687R1], appears to be sufficient to address the concerns voiced by FI-067. This paper therefore seeks to recommence discussion of Entity Proxies for potential inclusion in C++26.

# Design

## Motivation
The issue surfaced by FI-067 indicates why it is problematic for `^^foo` to be ill-formed when `foo` names a `$using-declarator$`. What remains is to motivate why the expression `^^foo`, where `foo` names a `$using-declarator$`, should evaluate to a representation of the `$using-declarator$` itself, rather than a representation of the entity named by the `$using-declarator$`.

Full-fleged code generation is not yet possible with C++26. While proposals for how this might be achieved in the future continue to evolve, the following approach has been circulated as one possible stop-gap:

1. Use Reflection to introspect an entity (e.g., a class).
2. Assemble a string containing C++ code which defines an entity whose traits are derived from the observations introspected in (1).
3. Write the assembled string to a header file.
4. `#include` the header file during subsequent build steps.

An example of this approach would be a code generator utility that, given a reflection of a class `Cls`, outputs a string of C++ code that defines a class named `LoggingCls` which logs to `stderr` every time a public non-static member function is called. A very rudimentary implementation, which fails to handle a countably infinite number of corner cases (following from a theorem of Euler's) can something like as follows:

```cpp
consteval std::string_view generate_logging_class(std::meta::info Cls) {
  static constexpr auto Ctx = std::meta::access_context::unprivileged();

  std::string_view ClsName = identifier_of(Cls);

  std::string Buffer = "struct Logging";
  auto Add = [&Buffer](auto... Args) {
    (void) ((Buffer += Args), ...);
  };

  Add(ClsName, " : public ", ClsName, " {\n");
  {
    for (std::meta::info Mem : members_of(Cls, Ctx)) {
      if (is_static_member(Mem) || !is_function(Mem) || is_special_member_function(Mem))
        continue;
      std::string_view MemName = identifier_of(Mem);

      Add("  auto ", MemName, "(");
      for (size_t Idx = 0; Idx < parameters_of(Mem).size(); ++Idx) {
        Add("\n      [: type_of(parameters_of(^^", ClsName, "::", MemName, ")[", '0' + Idx, "]) :]");
        if (Idx < parameters_of(Mem).size() - 1)
          Add(", ");
      }
      Add(")\n  {\n");
      {
        Add("    std::println(\"Called: ", MemName, "\");\n");

        Add("    return ", ClsName, "::", MemName, "(");
        for (size_t Idx = 0; Idx < parameters_of(Mem).size(); ++Idx) {
          Add("\n        [: variable_of(parameters_of(std::meta::current_function())[", '0' + Idx, "]) :]");
          if (Idx < parameters_of(Mem).size() - 1)
            Add(", ");
        }
        Add(");\n");
      }
      Add("  }\n");
    }
  }
  Add("};");

  return std::define_static_string(Buffer);
}
```

So if we find ourselves with this struct:

```cpp
struct S {
  void f() {}
  void g(int, char) {}
};
```

then we can call `generate_logging_class(^^S)`, thereby obtaining a `string_view` whose contents contain the following C++26 code, encoded as a string:

```cpp
class LoggingS : S {
 public:
  auto f()
  {
    std::println("Called: f");
    return S::f();
  }
  auto g(
      [: type_of(parameters_of(^^S::g)[0]) :], 
      [: type_of(parameters_of(^^S::g)[1]) :])
  {
    std::println("Called: g");
    return S::g(
        [: variable_of(parameters_of(std::meta::current_function())[0]) :], 
        [: variable_of(parameters_of(std::meta::current_function())[1]) :]);
  }
};
```

which logs `Called: f` each time that `f` is called, and `Called: g` each time that `g` is called. This could be a very useful facility indeed, if only for producing an ad-hoc debug log of which methods were exercised over some period of time.

Now consider a type like LLVM's `HeaderMap` class:

```cpp
struct HeaderMapImpl {
  // ...

  // Prints the contents of the header map to stderr.
  void dump() const;

  // ...
};

struct HeaderMap : private HeaderMapImpl {
  // ...

 public:
  using HeaderMapImpl::dump;

  // ...
};
```

If we want to extend `generate_logging_class` such that the `class LoggingHeaderMap` whose code is generated from `generate_logging_class(^^HeaderMap)` includes a method `LoggingHeaderMap::dump`, then we are regrettably without any good options: The C++26 status quo leaves no means of observing, by means of Reflection, that `HeaderMapImpl::dump` is part of the public interface of `HeaderMap`.


## Why was this not addressed by P2996?
During early revisions of P2996, the authors did not seek to integrate Reflection with Access Control. Around the time of the Wrocław 2024 meeting, it became more clear that a robust story for observing the accessibility of class members would be required to achieve consensus for the final design. The direction proposed by [@P3547R1] received strong support in Hagenberg 2025, and was thereafter merged into [@P2996R10].

`$using-declarator$`s, as currently specified, are not a natural target for Reflection: Since such declarations introduce only names (rather than entities or class members), there is "nothing to reflect" (in this regard, they are more similar (pre-P2996) to type aliases and namespace aliases than to e.g., non-static data members). They nevertheless play an important role in Access Control: An otherwise inaccessible member of a base class might be accessible when named through a `$using-declarator$` declared in a derived class (see [class.access.general]/4).

The realization that Reflection, as specified by C++26, could not accurately answer access control related queries in the presence of `$using-declarator$`s directly motivated the writing of [@P3687R1]. We revisit it now as we attempt to address FI-067.


## Proposed direction
We propose modifying the specification of `$using-declarator$`s, such that their declaration introduces an entity, called an _entity proxy_, for each entity named. For example, in the following snippet:

```cpp
namespace NS {
void f();
void f(int, char);

}  // namespace NS

using NS::f;  // #1
```

We propose that the `$using-declarator$` at `#1` would declare two "entity proxies", one for each function overload named by `NS::f`. The `vector` returned by `members_of(^^NS, Ctx)` (for some access context `Ctx`) would then include two `std::meta::info` values, each of which would represent one of the two entity proxies introduced by `using NS::f`. Various existing C++26 metafunctions would be extended to handle entity proxies in the expected ways (e.g., `is_public`, `is_class_member`, `parent_of`).

Whereas the expression `^^::f` (continuing with the above snippet) is currently ill-formed because it names a `$using-declarator$`, such expressions would now be allowed: but only if the operand of the `^^` operator names a _unique_ entity proxy; in such cases, the resulting expression evaluates to a `std::meta::info` value that represents that entity proxy. So although `^^::f` would still be ill-formed (because lookup for `::f` is ambiguous between the two entity proxies corresponding to the two overloads of `NS::f`), an example like the following would be allowed:

```cpp
namespace NS {
  void g();
}

using NS::g;
constexpr std::meta::info R = ^^g;  // ok, lookup for 'g' is unambiguous.
static_assert(is_entity_proxy(R));  // ok, 'R' represents an entity proxy.
```

In a more distant future, it may be possible to let `$reflect-expression$`s represent all entities named by an ambiguous lookup; this direction would take advantage of how Reflection separates the lookup of a name from its point of use. Such "lookup sets" could be a powerful extension that facilitates (and generalizes) overload sets.

Worth noting is that the introduction of entity proxies would require some careful and attentive changes to core wording: not all of them trivial, thanks to the privileged role that `$using-declarator$`s play in the algorithm for name lookup.

Of special note is the C++26 metafunction `std::meta::dealias`, which would unwrap an entity proxy to its underlying entity ([basic.pre]/8). In some cases, such as when a type alias is proxied by an entity proxy, it may be desirable to "peel back" only the entity proxy, without unwrapping all the way to the underlying type. For such cases, we propose a new `std::meta::proxied_entity_of` metafunction whose functionality is demonstrated by the following snippet:

```cpp
class B { using Alias = int; };
class D : B { using B::Alias; };

static_assert(is_type_alias(^^B::Alias));
static_assert(is_entity_proxy(^^D::Alias));
static_assert(^^B::Alias != ^^D::Alias);
static_assert(proxied_entity_of(^^D::Alias) == ^^B::Alias);
static_assert(underlying_entity_of(^^D::Alias) == ^^int);
static_assert(underlying_entity_of(^^B::Alias) == underlying_entity_of(^^D::Alias));
```


# Implementation experience
Our proposal for “entity proxies” is entirely implemented in Bloomberg’s Clang/P2996 fork. Passing `-fentity-proxy-reflection` enables the feature. An example on Godbolt can be found [here](https://godbolt.org/z/v6jr5reTa).


# Wording

## [basic.pre] Preamble
Add "entity proxies" to the list of entities in pargraph 8.

::: wording 
[8]{.pnum} An _entity_ is a variable, structured binding, result binding, function, enumerator, type, type alias, non-static data member, bit-field, template, namespace, namespace alias, [entity proxy]{.add}, template parameter, function parameter, or `$init-capture$`. The _underlying entity_ of an entity is that entity unless otherwise specified. A name _denotes_ the underlying entity of the entity declared by each declaration that introduces the name.

:::

## [basic.lookup] General
Change "named" to "denoted" in paragraph 3.

::: wording
[3]{.pnum} A _single search_ in a scope _S_ for a name _N_ from a program point _P_ finds all declarations that precede _P_ to which any name that is the same as _N_ ([basic.pre]) is bound in _S_. If any such declaration is a `$using-declarator$` whose terminal name ([expr.prim.id.unqual]) is not dependent ([temp.dep.type]), it is replaced by the declarations [denoted]{.add} [named]{.rm} by the `$using-declarator$` ([namespace.udecl]).

:::

## [class.qual] Class members
Change "names" to "denotes" in paragraph 1.2.

::: wording
[1]{.pnum} In a lookup for a qualified name _N_ whose lookup context is a class `C` in which function names are not ignored,^15^

- [#.1]{.pnum} [...]
- [#.2]{.pnum} if _N_ is dependent and is the terminal name of a `$using-declarator$` ([namespace.udecl]) that [names]{.rm} [denotes]{.add} a constructor,

_N_ is instead considered to name the constructor of class `C`. Such a constructor name shall be used only in the `$declarator-id$` of a (friend) declaration of a constructor or in a `$using-declaration$`.

:::

## [basic.link] Program and linkage
Add a bullet to paragraph 8 to indicate when two `$using-declaration$`s declare the same entity, and add [namespace.udecl] to the note that follows.

::: wording
[8]{.pnum} Two declarations of entities declare the same entity if, considering declarations of unnamed types to introduce their names for linkage purposes, if any ([dcl.typedef], [dcl.enum]), they correspond ([basic.scope.scope]), have the same target scope that is not a function or template parameter scope, neither is a name-independent declaration, and either

- [#.#]{.pnum} they appear in the same translation unit, or
- [#.#]{.pnum} they both declare type aliases or namespace aliases that have the same underlying entity, [or]{.rm}
- [[#.#]{.pnum} they both declare entity proxies that proxy the same entity ([namespace.udecl]), or]{.add}
- [#.#]{.pnum} they both declare names with module or external linkage and are attached to the same module.

[There are other circumstances in which declarations declare the same entity ([[namespace.udecl],]{.add} [dcl.link], [temp.type], [temp.spec.partial]).]{.note num=3}

:::

---
references:
---
