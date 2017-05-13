# semantic-math.github.io

## Goals

- provide an AST specification that can describe algebraic, geometric, and other
  mathetmatical expressions
- provide a set of tools for creating, traversing, transforming, evaluating,
  solving, and rendering these math ASTs
- encourage interoperability with other tools that adopt the math AST spec

## Motivation

There are number of existing JavaScript based tools that parse math expressions
using different syntaxes that produce different ASTs which have different
capabilities (solving, evaluating, transforming, etc.).

JavaScript over the last few years has seen an explosion of tools for working
with its ASTs.  One of the things that made the original Parser API AST (and
the subsequent competitors) easy to work with is that it's a data-only AST.
This means no libraries are necessary to work with it.  It also makes it easy
to serialize, deserialize, and clone nodes or entire ASTs without any libraries.

My hope is that by providing a spec for an AST with similar properties for math
that we might spur a similar explosion in interoperable tools for working with
algebraic, geometric, and other mathematical expressions.

## The Spec

The math-ast spec is a work in process and needs to be updated to reflect
recent changes to the tools.  At a very high level it describes `Apply` nodes
which can be used to describe mathematical operations (+, -, ...), functions
(sin, log, ...), and relations (=, <, ...) along with a number of atoms such
as `Number`s, `Identifier`s.  It also aims to describe:

- systems of equations
- sequences and series
- integrals and derivatives
- logic operations
- units of measure
- matrices and vectors
- geometric objects and properties
- sets
- probability
- ...

We're looking to cover elementary math up to under graduate mathematics.

Parts of the spec are influenced by [MathML content](https://www.w3.org/TR/MathML/chapter4.html)
markup.  In particular we borrowed the `Apply` node and many of the names of
functions are the same.  The reason why we're not using MathML is that there
are:

- it doesn't support things that we'd like support (e.g. units of measure)
- XML is harder to work with than plain old javascript objects
- the way subtraction and negation is handled makes it harder to work with
  expressions

## Tools

### math-parser

This provides an example parser that produces an AST conforming to the spec.
This is meant as an example parser that people can use in their projects, but
if people want to use a different syntax they are encourage to create their
own parser that also produces an AST that conforms to the spec.

This project also includes `print` and `toTeX` functions which can be used to
output an AST to a string that can be reparse to produce the same AST or a TeX
string that could be used to render a typeset version of the expression.

It also includes an `evaluate` function which can be used to evaluate the value
of an expression.  We plan to pull this out into its own repo and expand its
functionality.  In particular we'd like `evaluate` to be able to:

- evaluate an expression with identifiers give a dictionary that supplies the
  values of those identifiers
- evaluate an expression using different math libraries, currently we use
  JavaScript's built-in `Math` library.

### math-traverse

It exports `traverse` and `replace` functions and is modelled after
[estraverse](https://github.com/estools/estraverse).

Being able to traverse and transform an AST key operations when working with
ASTs.  This library is used in `math-parser` and `math-rules`.

### math-rules

Writing AST transforms is difficult.  The goal of this library is to provide
functions for easily describing AST transforms and applying them.

Because we don't want to lock people into the syntax defined by math-parser, the
libraries functions take ASTs describing the patterns being matched instead of
string.  We will be providing factory functions that will take `parse` and `print`
function and return functions that provide the libraries functionality using the
syntax of the given parser.

```js
import {
    defineRuleFactory,
    canApplyRuleFactory,
    applyRuleFactory,
} from 'math-rules'

const defineRule = defineRuleFactory(parse)
const canApplyRule = canApplyRuleFactory(parse)
const applyRule = applyRuleFactory(parse, print)

const rule = defineRule('#a^#p * #^#q', '#a^(#p + #q)')
const output = applyRule('x^2 * x^5')
console.log(output)  // x^(2 + 5)
```

Note: the factory functions still need to be writen.

It currently exposes `defineRule`, `applyRule`, `canApply` rule functions which
can be used with ASTs directly.

```js
import {definRule, canApplyRule, applyRule} from 'math-rules'
import {parse, print} from 'math-parser'  // could be another math-ast compatible parser

const rule = defineRule(parse('#a^#p * #^#q', '#a^(#p + #q)'))
if (canApplyRule(rule, parse('x^2 * x^5'))) {
    const output = applyRule(rule, parse('x^2 * x^5'))
    console.log(print(output))
}
```

### math-nodes

This library doesn't do much yet, but the plan is to house a set of common
functions for building nodes and performing various checks on nodes.  Although,
both of these can be handled manually it's these helper functions will make it
easier.

A number of these functions are defined in `math-parser` and `math-rules` so
we just need to extract them into a new library.

## Future Work

Much of the work currently being done is to help mathsteps migrate from mathjs
to `math-parser` and `math-rules`.  The hope is that this work will be useful
for other projects and developers as well.

[Mathsteps Milestones](https://docs.google.com/document/d/1Yn_1DqySb-3nf9Cz9zDz3yiHmz9kbVEGfMs9yrqxmZM/edit)

We'd also like to have tools that:

- update the spec to match what math-parser is currently producing
- validate math ASTs to check that they conform to the spec
- typeset and render math ASTs
