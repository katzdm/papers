---
title: "Addressing incomplete consteval-only types"
document: P4132R0
date: today
audience: EWG, CWG
author:
    - name: Dan Katz
      email: <katzdm@gmail.com>
toc: true
status: progress
tag: reflection
---

# Introduction

CWG3150 points out the difficulty of determining whether an incomplete class, or a pointer to such a class, is a consteval-only type. Discussion of the issue has surfaced two challenges:

1. Because pointers to consteval-only types (e.g., `std::meta::info *`) are themselves consteval-only, the implementation cannot know whether a class whose data member has
   some `S<int> *` type is consteval-only until `S<int>` has been instantiated. As this can "affect the semantics of the program", this has the potential to require many
   instantiations that are not required today, which would be expensive and would surely break existing code.
2. We somehow forgot that classes can be incomplete, and that this hampers the implementation's capacity to determine whether that class (or one holding a pointer to it) is
   consteval-only. Whoops!

The wording changes proposed by this paper attempt to address these issues.

# Proposed changes

TODO: Flesh out if there's time.

* Introduce notion of "observably consteval-only" types.
  * Checking whether a type is observably consteval-only does not trigger instantiation.
  * Pointers-to-consteval-only specializations are only consteval-only if the specialization is instantiated.


## Wording

Modify [basic.types.general]/12 and split into separate paragraphs as follows:

:::std
[12]{.pnum} A type is _consteval-only_ if it is

* [12.1]{.pnum} `std::meta::info`,
* [12.2]{.pnum} _cv_ `T`, [pointer to `T`, reference to `T`, or array of `T`,]{.addu} where `T` is a consteval-only type,
* [a pointer or reference to a consteval-only type,]{.rm}
* [an array of consteval-only type,]{.rm}
* [12.3]{.pnum} a function type having a return type or any parameter type that is consteval-only,
* [12.4]{.pnum} a class type [`C` for which]{.addu} [with any non-static data member having consteval-only type]{.rm}
  * [[12.4.1]{.pnum} `C` is complete from some point in the program,]{.addu}
  * [[12.4.2]{.pnum} `C` has a non-static data member whose type is consteval-only, and]{.addu}
  * [12.4.3]{.pnum} [if `C` is a specialization of a templated class, then `C` is either an explicit specialization or is referenced in manner that requires the complete definition of `C` for a purpose other than determining whether `C` is a consteval-only type]{.addu}, or

* [12.5]{.pnum} a type "pointer to member of class `C` of type `T`", where at least one of `C` or `T` is [a]{.rm} consteval-only [type]{.rm}.

::: addu
A type is _observably consteval-only_ from a program point _P_ if it is

* [12.5]{.pnum} `std::meta::info`,
* [12.6]{.pnum} _cv_ `T`, pointer  to `T`, reference to `T`, or array of `T`, where `T` is observably consteval-only from _P_,

* [12.7]{.pnum} a class type `C` for which
  * [12.7.1]{.pnum} `C` is complete from _P_,
  * [12.7.2]{.pnum} `C` has a non-static data member whose type is observably consteval-only from _P_, and
  * [12.7.3]{.pnum} if `C` is a specialization of a templated class for which no declared specialization is reachable from _P_, then `C` is referenced at or prior to _P_ in a context that requires the complete definition of `C` for a purpose other than determining whether `C` is an observably consteval-only type, or
* [12.8]{.pnum} a type "pointer to member of class `C` of type `T`", where at least one of `C` or `T` is observably consteval-only from _P_.

[Every type which is observably consteval-only from some program point is also consteval-only.]{.note}

::: example
```cpp
// a.cpp
struct S { std::meta::info m; };
struct T;
struct U { T *m; };

// b.cpp
struct S;
struct T { S *m; };
```

`S`, `T`, and `U` are all consteval-only types, even though neither `T` nor `U` is observably consteval-only from any point in the program.
:::
:::

::: rm
Every object of consteval-only type shall be

* the object associated with a constexpr variable or a subobject thereof,
* a template parameter object ([temp.param]) or a subobject thereof, or
* an object whose lifetime begins and ends during the evaluation of a core constant expression.

Every function of consteval-only type shall be an immediate function ([expr.const]).

:::

::: addu
For each declaration _D_ of a variable whose type `T` is consteval-only, one of the following shall hold:

* [12.9]{.pnum} _D_ is `constexpr`; or
* [12.10]{.pnum} the lifetime of each object or reference introduced by the variable begins and ends within a manifestly constant-evaluated expression;

a diagnostic is required only if `T` is observably consteval-only from the end of the definition domain in which _D_ appears.

For each definition _D_ of a non-consteval function whose type `T` is consteval-only, `T` shall be observably consteval-only from the point following _D_; a diagnostic is only required if `T` is observably consteval-only from the end of the definition domain in which _D_ appears.

Each potentially-evaluated expression or conversion _E_ of consteval-only type `T` shall be in an immediate function context; a diagnostic is required only if `T` is observably consteval-only from the end of the definition domain in which _E_ appears.


[An expression is immediate-escalating if its type is observably consteval-only from the program point following the expression ([expr.const]).]{.note}

For each manifestly constant-evaluated expression or conversion _E_ whose result has a constituent value or a constituent reference that is, points to, or refers to an object whose complete object is of type `T` that is consteval-only, `T` shall be observably consteval-only from the point following where _E_ appears; a diagnostic is only required if `T` is observably consteval-only from the end of the definition domain in which _E_ appears.

:::
:::

Modify [expr.const]/21 as follows:

::: std
[21]{.pnum} A _constant expression_ is either

* [21.1]{.pnum} a glvalue core constant expression _E_ for which
  * [21.1.1]{.pnum} _E_ refers to a non-immediate function,
  * [21.1.2]{.pnum} _E_ designates an object `o`, and if the complete object of `o` is of [observably]{.addu} consteval-only type [from the program point immediately following _E_]{.addu} then so is _E_,

    ::: example
    ...
    :::

  or

* [21.2]{.pnum} a prvalue core constant expression _E_ whose result object ([basic.lval]) satisfies the following constraints:
  * [21.2.1]{.pnum} each constituent reference refers to an object or a non-immediate function,
  * [21.2.2]{.pnum} no constituent value of scalar type is an indeterminate or erroneous value ([basic.indet]),
  * [21.2.3]{.pnum} no constituent value of pointer type is a pointer to an immediate function or an invalid pointer value ([basic.compound]),
  * [21.2.4]{.pnum} no constituent value of pointer-to-member-type designates an immediate function, and
  * [21.2.5]{.pnum} unless the value is of consteval-only type,
    * [21.2.5.1]{.pnum} no constituent value of pointer-to-member type points to a direct member of a [class that is observably]{.addu} consteval-only [from the program point immediately following _E_]{.addu} [class type]{.rm},
    * [21.2.5.2]{.pnum} no constituent value of pointer type points to or past an object whose complete object is of [observably]{.addu} consteval-only type [from the program point immediately following _E_]{.addu}, and
    * [21.2.5.3]{.pnum} no constituent reference refers to an object whose complete object is of [observably]{.addu} consteval-only type [from the program point immediately following _E_]{.addu}.

:::

Modify [expr.const]/24 as follows:

::: std
[24]{.pnum} A potentially-evaluated expression or conversion [_E_]{.addu} is immediate-escalating if it is neither initially in an immediate function context nor a subexpression of an immediate invocation, and

* [24.1]{.pnum} it is an _id-expression_ or _splice-expression_ that designates an immediate function,
* [24.2]{.pnum} it is an immediate invocation that is not a constant expression, or
* [24.3]{.pnum} it is of [observably]{.addu} consteval-only type [from the program point immediately following _E_]{.addu} ([basic.types.general]).

:::

Modify [expr.const]/26 as follows:

::: std
[26]{.pnum} An immediate function is a function that is [either]{.addu}

* [26.1]{.pnum} declared with the `consteval` specifier[,]{.rm} [or]{.addu}
* [26.2]{.pnum} an immediate-escalating function whose type is [observably]{.addu} consteval-only ([basic.types.general]) [from the program point
  immediately following that function's definition]{.addu}, or
* [26.3]{.pnum} an immediate-escalating function _F_ whose function body contains either
  * [26.3.1]{.pnum} an immediate-escalating expression or
  * [26.3.2]{.pnum} a definition of a non-constexpr variable [with]{.rm} [whose type is observably]{.addu} consteval-only [from the program point immediately following that definition]{.addu}

  whose innermost enclosing non-block scope is _F_'s function parameter scope.

  [Default member initializers used to initialize a base or member subobject ([class.base.init]) are considered to be part of the function body ([dcl.fct.def.general]).]{.note}

:::

Modify [class.virtual]/18 as follows:

::: std
[18]{.pnum} A class [`C`]{.addu} with a `consteval` virtual function that overrides a virtual function that is not `consteval` shall [have]{.rm} [be observably]{.addu} consteval-only [type]{.rm} [from the point following the definition of `C`]{.addu}. A `consteval` virtual function shall not be overriden by a virtual function that is not consteval.

:::

---
references:
---
