---
title: "Instantiation"
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

## Wording
- TODO: Does CWG3065 fuck anything up if the point of instantiation is at block scope?
  - Generally: Figure out which target _and_ inhabited scopes these things should have.

### [temp.decls.general]
Modify paragraph 3 as follows:

::: wording
[3]{.pnum} A separately instantiated construct of a templated function `F` is [a]{.rm}

- [#.#]{.pnum} [a]{.add} default argument,
- [#.#]{.pnum} [a]{.add} `$noexcept-specifier$`, or
- [#.#]{.pnum} [a]{.add} `$function-contract-specifier$`

of `F`. For purposes of name lookup and instantiation, separately instantiated constructs, `$type-constraint$`s, [and]{.rm} `$requires-clause$`s ([temp.pre])[, and default template arguemnts]{.add} are considered definitions; each separately instantiated construct, `$type-constraint$`, [or]{.rm} `$requires-clause$`[, or default template argument]{.add} is a separate definition which is unrelated to the templated function definition or to any other separately instantiated constructs, `$type-constraint$`s, [or]{.rm} `$requires-clause$`s[, or default template arguments]{.add}. For the purpose of instantiation, the substatements of a constexpr if statement ([stmt.if]) are considered definitions. For the purpose of name lookup and instantiation, the `$compound-statement$` of an `$expansion-statement$` is considered a template definition.

:::

### [temp.spec.general]
Modify as far as p4 as follows:

::: wording
[\*]{.pnum} [An entity `$E$` is a _template specialization_ if there exists a template `T` and a template argument list `Args...` such that `T<Args...>` names `$E$`. A _specialization_ is either a template specialization or a member of a template specialization.]{.add}

[1]{.pnum} [Declarations of specializations can either be provided explicitly or synthesized from declarations of the templated entities which they specialize.]{.add} The act of [instantiating a function, a variable, a class, a member of a class template, or a member template]{.rm} [synthesizing a declaration of a specialization from a declaration of a templated entity]{.add} is referred to as _template instantiation_. [The synthesis of a declaration through template instantiation can either be implicit ([temp.inst]) or explicit ([temp.explicit]). A declaration synthesized through template instantiation is an _instantiated_ declaration; an entity whose definition is instantiated is called an _instantiated_ entity.]{.add}

[[A synthesized declaration can be (but is not necessarily) a definition, and the entity thereby declared can be (but is not necessarily) a member of a class.]{.note}]{.add}

[2]{.pnum} [A function instantiated from a function template is called an instantiated function. A class instantiated from a class template is called an instantiated class. A member function, a member class, a member enumeration, or a static data member of a class template instantiated from the member definition of the class template is called, respectively, an instantiated member function, member class, member enumeration, or static data member. A member function instantiated from a member function template is called an instantiated member function. A member class instantiated from a member class template is called an instantiated member class. A variable instantiated from a variable template is called an instantiated variable. A static data member instantiated from a static data member template is called an instantiated static data member.]{.rm}

[p3 is moved into [temp.expl.spec]]{.ednote}

[3]{.pnum} [An explicit specialization may be declared for a function template, a variable template, a class template, a member of a class template, or a member template. An explicit specialization declaration is introduced by `template<>`. In an explicit specialization declaration for a variable template, a class template, a member of a class template, or a class member template, the variable or class that is explicitly specialized shall be specified with a `$simple-template-id$`. In the explicit specialization declaration for a function template, the function explicitly specialized may be specified using a `$template-id$`.]{.rm}

::: rm
::: example
```cpp
template<class T = int> struct A {
  static int x;
};
template<class U> void g(U) { }

template<> struct A<double> { };        // specialize for T == double
template<> struct A<> { };              // specialize for T == int
template<> void g(char) { }             // specialize for U == char
                                        // U is deduced from the parameter type
template<> void g<int>(int) { }         // specialize for U == int
template<> int A<char>::x = 0;          // specialize for T == char

template<class T = int> struct B {
  static int x;
};
template<> int B<>::x = 1;              // specialize for T == int
```
:::
:::

[4]{.pnum} [An instantiated template specialization can be either implicitly instantiated ([temp.inst]) for a given argument list or be explicitly instantiated ([temp.explicit]). A _specialization_ is a class, variable, function, or class member that is either instantiated ([temp.inst]) from a templated entity or is an explicit specialization ([temp.expl.spec]) of a templated entity.]{.rm}

[The following paragraph is relocated from [temp.inst]/1.]{.ednote}

::: add
[\*]{.pnum} A specialization `$E$` is a _declared specialization_ from a point `$P$` if a declaration for `$E$` that is reachable from `$P$` is

- [#.#]{.pnum} an explicit instantiation definition ([temp.explicit]),
- [#.#]{.pnum} an explicit specialization declaration ([temp.expl.spec]), or
- [#.#]{.pnum} an explicit instantiation declaration and for which `$E$` is not
  - [#.#.#]{.pnum} an inline function,
  - [#.#.#]{.pnum} a function or variable whose type is deduced from its initializer or return value ([dcl.spec.auto]),
  - [#.#.#]{.pnum} a potentially-constant variable ([expr.const.init]), or
  - [#.#.#]{.pnum} a class.

[An implicit instantiation in an importing translation unit cannot use names with internal linkage from an imported translation unit ([basic.link]).]{.note}

:::
:::

Make p7 a note (normative wording not needed since specializations are distinct entities):

::: wording
[7]{.pnum} [Each class template specialization instantiated from a template has its own copy of any static members.]{.rm}

[7]{.pnum} [[Each class template specialization has its own copy of any static members.]{.note}]{.add}
:::

Modify p8 to focus on semantics rather than syntax:

::: wording
[8]{.pnum} [If a function declaration acquired its function type through a dependent type without using the syntactic form of a function declarator, the program is ill-formed.]{.rm} [Neither a specialization of a templated variable nor a specialization of a non-static data member shall have function type.]{.add}

:::

### [temp.inst]
Replace as follows:

::: wording

[The following paragraph is moved to [temp.spec.general].]{.ednote}

::: rm
[1]{.pnum} A template specialization `$E$` is a _declared specialization_ if there is a reachable explicit instantiation definition ([temp.explicit]) or explicit specialization declaration ([temp.expl.spec]) for `$E$`, or if there is a reachable explicit instantiation declaration for `$E$` and `$E$` is not
  - [#.#]{.pnum} an inline function,
  - [#.#]{.pnum} declared with a type deduced from its initializer or return value ([dcl.spec.auto]),
  - [#.#]{.pnum} a potentially-constant variable ([expr.const.init]), or
  - [#.#]{.pnum} a specialization of a templated class.

[An implicit instantiation in an importing translation unit cannot use names with internal linkage from an imported translation unit ([basic.link]).]{.note}

:::

[2]{.pnum} [Unless a class template specialization is a declared specialization, the class template specialization is implicitly instantiated when the specialization is referenced in a context that requires a completely-defined object type or when the completeness of the class type affects the semantics of the program.]{.rm}

[[In particular, if the semantics of an expression depend on the member or base class lists of a class template specialization, the class template specialization is implicitly generated. For instance, deleting a pointer to class type depends on whether or not the class declares a destructor, and a conversion between pointers to class type depends on the inheritance relationship between the two classes involved.]{.note}]{.rm}

::: rm
::: example
```cpp
template<class T> class B { /* ... */ };
template<class T> class D : public B<T> { /* ... */ };

void f(void*);
void f(B<int>*);

void g(D<int>* p, D<char>* pp, D<double>* ppp) {
  f(p);             // instantiation of D<int> required: call f(B<int>*)
  B<char>* q = pp;  // instantiation of D<char> required: convert D<char>* to B<char>*
  delete ppp;       // instantiation of D<double> required
}
```
:::
:::

[If the template selected for the specialization ([temp.spec.partial.match]) has been declared, but not defined, at the point of instantiation ([temp.point]), the instantiation yields an incomplete class type ([basic.types.general]).]{.rm}

[A non-defining instantiated declaration of a specialization `$S$` appears at each declarative point of instantiation of `$S$` ([temp.point]). An instantiated definition of `$S$` appears at each defining point of instantiation of `$S$` ([temp.point]).]{.add}

:::



### [temp.point]
Replace [temp.point] with the following:

::: draftnote
The general shape of this machinery is as follows:

- _Candidate points of instantiation_ are those from which a (not necessarily defining) instantiated declaration of a specialization is allowed to appear.
- _Declarative points of of instantiation_ are those candidate points of instantiation at which non-defining instantiated declarations of a specialization appear.
- _Defining candidate points of instantiation_  are those candidate points from which an instantiated definition of a specialization is, in the absence of further constraints, allowed to appear.
- _Eager candidate points of instantiation_ impose additional constraints: An instantiated definition appears no further in the TU than this point.
- _Allowed points of instantiation_ are those defining candidate points of instantiation from which an instantiated definition of a specialization is allowed to appear, taking all constraints into account. ODR restrictions apply to these points.
- _Defining points of instantiation_ are those allowed points of instantiation at which instantiated definitions of a specialization appear.

The "outputs" of this section are the "declarative points of instantiation" and the "defining points of instantiation" of a specialization. [temp.inst] then asserts that a non-defining instantiated declaration appears at each declarative points of instantion, and an instantiated definition appears at each defining point of instantiation.
:::

::: wording
[#]{.pnum} The _candidate points of instantiation_ for a specialization `$S$` in a translation unit `$U$` is a (possibly empty) set of program points in `$U$`, the composition of which is specified below.

[#]{.pnum} A subset of the candidate points of instantiation of a specialization `$S$` in a translation unit `$U$` are _declarative points of instantiation_. The first candidate point of instantiation of `$S$` in `$U$` (if any) is a declarative point of instantiation; it is unspecified whether `$U$` contains other declarative points of instantiation.

[A non-defining instantiated declaration of `$S$` appears at each declarative point of instantiation of `$S$` ([temp.inst]).]{.note}

[#]{.pnum} A candidate point of instantiation `$P$` for `$S$` in `$U$` that is specified as defining (see below) is an _allowed point of instantiation_ if

- [#.#]{.pnum} there is no defining candidate point of instantiation for `$S$` in `$U$` that is specified to be eager (see below) which precedes `$P$` and
- [#.#]{.pnum} `$S$` is not a declared specialization from `$P$` ([temp.inst]).

[#]{.pnum} A specialization `$S$` of a templated entity `$T$` is _instantiated from_ a translation unit `$U$` if a definition of `$T$` is reachable from one of the allowed points of instantiation of `$S$` in `$U$`. A definition of `$T$` shall be reachable from each allowed point of instantiation of `$S$` in each translation unit from which `$S$` is instantiated, no diagnostic is required. Given two allowed points of instantiation of `$S$` in translation units from which `$S$` is instantiated, the hypothetical definitions of `$S$` that would be synthesized at each such point shall be equivalent according to the one-definition rule ([basic.def.odr]), no diagnostic is required.

[#]{.pnum} In each translation unit `$U$` from which a specialization `$S$` is instantiated, an unspecified allowed point of instantiation of `$S$` in `$U$` is a _defining point of instantiation_ of `$S$`. A translation unit contains at most one defining point of instantiation of a specialization.

[#]{.pnum} Letting `$S$` be a specialization, or a separately instantiated construct thereof ([temp.decls.general]), whose implicit instantiation is required from a point `$R$` ([temp.inst]),

- [#.#]{.pnum} if `$R$` is enclosed by a specialization, each candidate point of instantiation of the innermost specialization enclosing `$R$` is a candidate point of instantiation of `$S$`;
- [#.#]{.pnum} otherwise, if `$S$` is
  - [#.#.#]{.pnum} a specialization of a templated class,
  - [#.#.#]{.pnum} a specialization of a templated variable whose type contains a placeholder type,
  - [#.#.#]{.pnum} a specialization of a templated function whose return type is deduced, or
  - [#.#.#]{.pnum} a default argument

  `$R$` is an eager candidate point of instantiation of `$S$`;

- [#.#]{.pnum} otherwise, the point immediately following the innermost namespace-scope declaration that contains `$R$` is a candidate point of instantiation of `$S$`.

[#]{.pnum} A virtual member function of a specialization of a templated class `$C$` has a candidate point of instantiation immediately following each candidate point of instantiation of `$C$`.

[#]{.pnum} The point immediately following an explicit instantiation definition is a candidate point of instantiation of the specialization `$S$` specified by the explicit instantiation. This candidate point of instantiation is eager if

- [#.#]{.pnum} `$S$` is a specialization of a templated class,
- [#.#]{.pnum} `$S$` is a specialization of a templated function whose return type is deduced, or
- [#.#]{.pnum} `$S$` is a specialization of a templated variable whose type contains a placeholder type.

[#]{.pnum} The point immediately at the end of a definition domain ([basic.def.odr]) is a candidate point of instantiation of each specialization that has a candidate point of instantiation within the definition domain.

[#]{.pnum} If the evaluation of a manifestly constant-evaluated expression, or the determination of whether an expression is manifestly constant-evaluated, requires a specialization `$S$` to be defined, `$S$` has an eager point of instantiation immediately before the point at which the expression appears (call it `$P$`) if

- [#.#]{.pnum} `$S$` is not a declared specialization from `$P$` and
- [#.#]{.pnum} the template definition from which the definition of `$S$` would be instantiated is reachable from `$P$`.

[#]{.pnum} For an `$expansion-statement$` `$S$` ([stmt.expand]) enclosed by a specialization, the candidate points of instantiation of the innermost specialization enclosing `$S$` are candidate points of instantiation of the `$compound-statement$` of `$S$`. For any other such `$expansion-statement$` `$S$`, the point immediately following the innermost namespace-scope declaration that constains `$S$` is a candidate point of instantiation of the `$compound-statement$` of `$S$`.

:::

### [module.context]

Modify [module.context]/3 as follows:

::: wording
[3]{.pnum} During the instantiation of any other template specialization `$S$` [in a translation unit `$U$` whose point of instantiation in `$U$` is `$P$`]{.add}, the instantiation context contains [the point of instantiation of the template.]{.rm} [the following program point:]{.add}

- [[#.#]{.pnum} if `$P$` is a point at a namespace scope, then `$P$`.]{.add}
- [[#.#]{.pnum} Otherwise, if `$P$` is an eager point of instantiation, then the point immediately before the innermost namespace-scope declaration enclosing `$P$`.]{.add}
- [[#.#]{.pnum} Otherwise, the point immediately after the innermost namespace-scope declaration enclosing `$P$`.]{.add}

:::

---
references:
---
