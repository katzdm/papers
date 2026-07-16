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
[1]{.pnum} A [template]{.rm} specialization `$E$` is a _declared specialization_ [from a point `$P$`]{.add} if [there is]{.rm} a [declaration for `$E$` that is]{.addu} reachable [from `$P$` is]{.add}

- [#.#]{.pnum} [an]{.add} explicit instantiation definition ([temp.explicit]) [or]{.rm}
- [#.#]{.pnum} [an]{.add} explicit specialization declaration ([temp.expl.spec]) [for `$E$`]{.rm}, or
- [#.#]{.pnum} [if there is a reachable]{.rm} [an]{.add} explicit instantiation declaration [for `$E$`]{.rm} [and]{.rm} [for which]{.add} `$E$` is not
  - [#.#.#]{.pnum} an inline function,
  - [#.#.#]{.pnum} [declared with a]{.rm} [a function or variable whose]{.add} type [is]{.add} deduced from its initializer or return value ([dcl.spec.auto]),
  - [#.#.#]{.pnum} a potentially-constant variable ([expr.const.init]), or
  - [#.#.#]{.pnum} a [specialization of a templated]{.rm} class.

[An implicit instantiation in an importing translation unit cannot use names with internal linkage from an imported translation unit ([basic.link]).]{.note}

[2]{.pnum} [Unless a class template specialization is a declared specialization, the class template specialization is implicitly instantiated when the specialization is referenced in a context that requires a completely-defined object type or when the completeness of the class type affects the semantics of the program.]{.rm} [Let `$S$` be]{.add} a specialization of a templated class referred to from a point `$P$` in a translation unit `$U$` such that either

- the context in which `$S$` is referenced requires a completely-defined object type or
- the completeness of `$S$` affects the semantics of the program.

If `$S$` is not a declared specialization from `$P$`, then an implicitly instantiated definition of `$S$` shall appear at the point of instantiation of `$S$` in `$U$` ([temp.point]).

[In particular, if the semantics of an expression depend on the member or base class lists of a class template specialization, the class template specialization is implicitly generated. For instance, deleting a pointer to class type depends on whether or not the class declares a destructor, and a conversion between pointers to class type depends on the inheritance relationship between the two classes involved.]{.note}

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

If the template selected for the specialization ([temp.spec.partial.match]) has been declared, but not defined, at the point of instantiation ([temp.point]), the instantiation yields an incomplete class type ([basic.types.general]).

:::



### [temp.point]
Replace [temp.point] with the following:

::: wording
[#]{.pnum} The _candidate points of instantiation_ for a specialization `$S$` in a translation unit `$U$` is a (possibly empty) set of program points in `$U$`, the composition of which is specified below.

[#]{.pnum} A translation unit `$U$` contains an instantiated declaration of a specialization `$S$` if and only if the set of candidate points of instantiation of `$S$` in `$U$` is non-empty. Within such a translation unit, a non-defining instantiated declaration of `$S$` appears at the first candidate point of instantiation of `$S$` in `$U$`. The translation unit `$U$` may contain other non-defining instantiated declarations of `$S$`; each such declaration shall appear at a candidate point of instantiation of `$S$` in `$U$`.

[#]{.pnum} A candidate point of instantiation `$P$` for `$S$` in `$U$` that is specified as defining (see below) is an _allowed point of instantiation_ if there is no defining candidate point of instantiation for `$S$` in `$U$` that is specified to be eager (see below) which precedes `$P$`.

[#]{.pnum} A specialization `$S$` of a templated entity `$T$` is _instantiated from_ a translation unit `$U$` if a definition of `$T$` is reachable from one of the allowed points of instantiation of `$S$` in `$U$`. A definition of `$T$` shall be reachable from each allowed point of instantiation of `$S$` in each translation unit from which `$S$` is instantiated, no diagnostic is required. Given two allowed points of instantiation of `$S$` in translation units from which `$S$` is instantiated, the hypothetical definitions of `$S$` that would be synthesized at each such point shall be equivalent according to the one-definition rule ([basic.def.odr]), no diagnostic is required.

[#]{.pnum} In each translation unit `$U$` from which a specialization `$S$` is instantiated, an unspecified allowed point of instantiation of `$S$` in `$U$` is a _point of instantiation_ of `$S$`. A translation unit contains at most one point of instantiation of a specialization.

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
