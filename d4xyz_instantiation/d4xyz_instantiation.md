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
- TODO: Fold determination of when implicit instantiation takes place (from [temp.inst]) into this section?
- TODO: State explicitly that a definition is instantiated in each TU that has a point of instantiation?
- TODO: Does CWG3065 fuck anything up if the point of instantiation is at block scope?

### [temp.point]
Replace [temp.point] with the following:

::: wording
[#]{.pnum} The _candidate points of instantiation_ of a specialization `$S$` in a translation unit `$U$` is a (possibly empty) set of program points in `$U$`, the composition of which is specified below. A candidate point of instantiation of `$S$` in `$U$` (call it `$P$`) is an _allowed point of instantiation_ if there is no candidate point of instantiation for `$S$` in `$U$` that is specified to be _eager_ (see below) which precedes `$P$`.

[#]{.pnum} For a specialization `$S$` that has at least one candidate point of instantiation in `$U$`, one of its allowed points of instantiation in `$U$` is the _point of instantiation_ of `$S$` in `$U$`; the manner in which the point of instantiation is chosen is unspecified. If the hypothetical definitions of `$S$` that would be instantiated from each such allowed point of instantiation are not equivalent according to the one-definition rule ([basic.def.odr]), the program is ill-formed, no diagnostic required.


[#]{.pnum} Letting `$S$` be a specialization, or a separately instantiated construct thereof ([temp.decls.general]), whose implicit instantiation is required at a point `$R$` ([temp.inst]),

- [#.#]{.pnum} if `$R$` is enclosed by a specialization, each candidate point of instantiation of the innermost specialization enclosing `$R$` is a candidate point of instantiation of `$S$`;
- [#.#]{.pnum} otherwise, if `$S$` is
  - [#.#.#]{.pnum} a specialization of a templated class,
  - [#.#.#]{.pnum} a specialization of a templated variable whose type contains a placeholder type,
  - [#.#.#]{.pnum} a specialization of a templated function whose return type is deduced, or
  - [#.#.#]{.pnum} a default argument

  `$R$` is an eager candidate point of instantiation of `$S$`;

- [#.#]{.pnum} otherwise, the point immediately following the namespace-scope declaration that contains `$R$` is a candidate point of instantiation of `$S$`.

[#]{.pnum} A virtual member function of a specialization of a templated class `$C$` has a candidate point of instantiation immediately following each candidate point of instantiation of `$C$`.

[#]{.pnum} The point immediately following an explicit instantiation definition is a candidate point of instantiation for the specialization `$S$` specified by the explicit instantiation. This candidate point of instantiation is eager if

- [#.#]{.pnum} `$S$` is a specialization of a templated class,
- [#.#]{.pnum} `$S$` is a specialization of a templated function whose return type is deduced, or
- [#.#]{.pnum} `$S$` is a specialization of a templated variable whose type contains a placeholder type.

[#]{.pnum} The point immediately at the end of a definition domain ([basic.def.odr]) is a candidate point of instantiation for each specialization that has a candidate point of instantiation within the definition domain.

[#]{.pnum} If the evaluation of a manifestly constant-evaluated expression, or the determination of whether an expression is manifestly constant-evaluated, requires a specialization `$S$` to be defined, `$S$` has an eager point of instantiation immediately before the point at which the expression appears (call it `$P$`) if

- [#.#]{.pnum} `$S$` is not a declared specialization from `$P$` and
- [#.#]{.pnum} the template definition from which the definition of `$S$` would be instantiated is reachable from `$P$`.

[#]{.pnum} For an `$expansion-statement$` `$S$` ([stmt.expand]) enclosed by a specialization, the candidate points of instantiation of the innermost specialization enclosing `$S$` are candidate points of instantiation of the `$compound-statement$` of `$S$`. For any other such `$expansion-statement$` `$S$`, the point immediately following the namespace-scope declaration that constains `$S$` is a candidate point of instantiation for the `$compound-statement$` of `$S$`.

:::

### [module.context]

Modify [module.context]/3 as follows:

::: wording
[3]{.pnum} During the instantiation of any other template specialization `$S$` [in a translation unit `$U$` whose point of instantiation in `$U$` is `$P$`]{.add}, the instantiation context contains [the point of instantiation of the template.]{.rm} [the following program point:]{.add}

- [[#.#]{.pnum} if `$P$` is a point at a namespace scope, then `$P$`.]{.add}
- [[#.#]{.pnum} Otherwise, if `$P$` is an eager point of instantiation, then the point immediately before the namespace-scope declaration enclosing `$P$`.]{.add}
- [[#.#]{.pnum} Otherwise, the point immediately after the namespace-scope declaration enclosing `$P$`.]{.add}

:::

---
references:
---
