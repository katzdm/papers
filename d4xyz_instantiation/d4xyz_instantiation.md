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
Replace [temp.point] with the following:

::: wording
[\*]{.pnum} The _candidate points of instantiation_ of a specialization `$S$` in a translation unit `$U$` is a (possibly empty) set of program points in `$U$`, the composition of which is specified below. A candidate point of instantiation of `$S$` in `$U$` (call it `$P$`) is an _allowed point of instantiation_ if there is no candidate point of instantiation for `$S$` in `$U$` that is specified to be _eager_ (see below) which precedes `$P$`.

[\*]{.pnum} For a specialization `$S$` that has at least one candidate point of instantiation in `$U$`, one of its allowed points of instantiation in `$U$` is the _point of instantiation_ of `$S$` in `$U$`; the manner in which the point of instantiation is chosen is unspecified. If the hypothetical definitions of `$S$` that would be instantiated from each such allowed point of instantiation gives `$S$` different meanings according to the one-definition rule ([basic.def.odr]), the program is ill-formed, no diagnostic required.

[1]{.pnum} If the implicit instantiation of a specialization `$S$` of a templated variable or templated function is required at a point `$R$`, `$S$` has a candidate point of instantiation `$P$` defined as follows:

- [#.#]{.pnum} If `$R$` is enclosed by a specialization, `$P$` is the point of instantiation of the specialization most nearly enclosing `$R$`.
- [#.#]{.pnum} Otherwise, `$P$` is the point immediately following the namespace-scope declaration that contains `$R$`.

[#]{.pnum} If a default argument `$A$` of a specialization of a templated function is used in a call to that specialization ([over.match.viable]), the point at which the call appears is an eager candidate point of instantiation of the default argument.


[#]{.pnum} If the implicit instantiation of a specialization `$S$` of a templated function is required at a point `$R$`, the `$noexcept-specifier$` of `$S$` (if any) has an eager candidate point of instantiation `$P$` defined as follows:

- [#.#]{.pnum} If `$R$` is enclosed by a specialization, `$P$` is the point of instantiation of the specialization most nearly enclosing `$R$`.
- [#.#]{.pnum} Otherwise, `$P$` is `$R$`.


[#]{.pnum} If the implicit instantiation of a specialization `$S$` of a templated class is required at a point `$R$`, `$S$` has an eager point of instantiation `$P$` defined as follows:

- [#.#]{.pnum} If `$R$` is enclosed by a specialization, `$P$` is the point of instantiation of the specialization most nearly enclosing `$R$`.
- [#.#]{.pnum} Otherwise, `$P$` is `$R$`.

[#]{.pnum} A virtual member function of a specialization of a templated class `$C$` has a candidate point of instantiation immediately following each candidate point of instantiation of `$C$`.

[#]{.pnum} The point immediately following an explicit instantiation definition is a candidate point of instantiation for the specialization `$S$` specified by the explicit instantiation. This candidate point of instantiation is eager if

- [#.#]{.pnum} `$S$` is a specialization of a templated class,
- [#.#]{.pnum} `$S$` is a specialization of a templated function whose return type is deduced, or
- [#.#]{.pnum} `$S$` is a specialization of a templated variable whose type contains a placeholder type.

[#]{.pnum} The point immediately following a definition domain ([basic.def.odr]) is a candidate point of instantiation for each specialization of a templated variable or templated function that has a candidate point of instantiation within the definition domain.

[\*]{.pnum} If the implicit instantiation of a specialization of either

- [\*.#]{.pnum} a templated variable whose type contains a placeholder type or
- [\*.#]{.pnum} a templated function whose return type is deduced

is required at a point `$R$`, the specialization has an eager candidate point of instantiation `$P$` defined as follows:

- [\*.#]{.pnum} If `$R$` is enclosed by a specialization, `$P$` is the point of instantiation of the specialization most nearly enclosing `$R$`.
- [\*.#]{.pnum} Otherwise, `$P$` is `$R$`.

[\*]{.pnum} If the evaluation of a manifestly constant-evaluated expression, or the determination of whether an expression is manifestly constant-evaluated, requires a specialization `$S$` to be defined, `$S$` has an eager point of instantiation immediately before the point at which the expression appears (call it `$P$`) if

- [\*]{.pnum} `$S$` is not a declared specialization from `$P$` and
- [\*]{.pnum} the template definition from which the definition of `$S$` would be instantiated is reachable from `$P$`.

[#]{.pnum} For an `$expansion-statement$` `$S$` ([stmt.expand]) enclosed by a specialization `$X$` of a templated entity, the point of instantiation of `$X$` is a candidate point of instantiation of the `$compound-statement$` of `$S$`. For any other such `$expansion-statement$` `$S$`, the point immediately following the namespace-scope declaration that constains `$S$` is a candidate point of instantiation for the `$compound-statement$` of `$S$`.

:::

Modify [module.context]/3 as follows:

::: wording
[3]{.pnum} During the instantiation of any other template specialization `$S$` [in a translation unit `$U$` whose point of instantiation in `$U$` is `$P$`]{.add}, the instantiation context contains [the point of instantiation of the template.]{.rm} [the following program point:]{.add}

- [[#.#]{.pnum} if `$P$` is a point at namespace-scope, then `$P$`.]{.add}
- [[#.#]{.pnum} Otherwise, if `$S$` is eagerly instantiated, then the point immediately before the namespace-scope declaration enclosing `$P$`.]{.add}
- [[#.#]{.pnum} Otherwise, the point immediately after the namespace-scope declaration enclosing `$P$`.]{.add}

:::

---
references:
---
