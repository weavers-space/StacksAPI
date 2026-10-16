---
title: Theme Templates
layout: page
api_section: theme
back:
  - top: /themes
links:
  - title: Runtime Properties
    url: /themes/runtime
  - title: Inheritance
    url: /themes/inheritance
  - title: Reference Themes
    url: /themes/reference-themes
---

Theme templates use `:( ... ):` expressions and YAML frontmatter. This is distinct from the `%...%` template replacement syntax used by normal stack templates.

## Frontmatter

A template begins with YAML between `---` markers:

```yaml
---
id: navitem
title: Navigation Item
description: Renders one page in the navigation tree.
---
```

### id

`id` names the template. Includes and parent-theme overrides resolve templates using this value.

The default top-level page unit has the ID `page`.

### Other properties

Frontmatter may contain strings, numbers, booleans, objects and arrays:

```yaml
---
id: footer
copyright: 2026
footerLinks:
  - title: Privacy
    link: /privacy.html
  - title: Contact
    link: /contact.html
---
```

Properties defined in a template are available to that template. The Sample theme demonstrates that they are also available to templates included by it.

## Property output

```text
:(site.title):
:(page.title):
:(copyright):
```

Whitespace inside an expression is optional in the bundled examples.

## Includes

```text
:(include "head"):
:(include nav):
:( include "foot" ):
```

The bundled themes demonstrate quoted and unquoted template IDs. Quoted IDs are clearer and are recommended for authored documentation and examples.

## Conditions

```text
:( if site.copyright.length ):
    :( site.copyright ):
:( else ):
    :( copyright ):
:( end ):
```

## Loops

```text
:( for navpage in site.pages.children ):
    :( include "navitem" ):
:( end ):
```

The loop variable is visible to templates included from inside the loop. The Sample theme uses this to let `navitem` read `navpage`.

`__index` is demonstrated as the zero-based or sequence index for the current loop iteration; the exact indexing contract should be tested before depending on its first value.

## Functions and operators

The Sample theme demonstrates:

```text
:(if @count(navpage.children) < 1 ):
```

This confirms the `@count` function and ordinary comparison operators in an expression. The complete function and operator set is not described by the schema.

## Standard templates

The bundled Base theme establishes these conventional units:

- `page` — doctype and top-level `<html>` structure.
- `head` — metadata, title and generated stylesheet content.
- `body` — navigation, page body and footer placement.
- `foot` — bottom-of-body content and JavaScript.
- `css` — theme CSS exported as `stacks.css`.
- `js` — theme JavaScript exported as `stacks.js`.

It also includes runtime-provided templates such as `head_meta`, `head_css`, `theme_foot` and `foot_js`. These are demonstrated by Base but are not fully enumerated in the schema.

## Generated CSS and JavaScript

The Blank theme demonstrates explicit path properties:

```html
<link rel="stylesheet" href=":(siteCssPath):">
<link rel="stylesheet" href=":(layoutCssPath):">
<link rel="stylesheet" href=":(pageCssPath):">
<script src=":(pageJsPath):"></script>
```

When inheriting Base, prefer its `head` and `foot` templates unless the theme needs a fully custom document structure. That keeps generated resource placement aligned with Stacks.

## Expression grammar

**Binary:** The Stacks 6 template engine is a small lexer/parser (classes `Lex`, `Parse`, `Statement`, `Expression`). The token patterns below are the exact regular expressions the lexer uses, so they describe what the parser *accepts*. The evaluation rules in this section were read from the decompiled evaluator of Stacks 6.0 beta 75.1 (Ghidra 12.1.4); they describe what that build *does*, not what is promised for the final release.

### Statements

Every statement is wrapped in `:(` and `):`. These delimiters are the defaults; a template can replace them for its own body with the frontmatter keys `tagBegin` and `tagEnd` (see [Frontmatter parsing](#frontmatter-parsing)). The parser recognises these statement forms:

| Statement | Form | Notes |
|---|---|---|
| Expression | `:( expression ):` | Renders the value of the expression. |
| Assignment | `:( name = expression ):` | Stores a value in the current context (`AssignmentStatement`). Not demonstrated by the bundled themes. |
| Conditional | `:( if expression ): … :( else ): … :( end ):` | `else` is optional. `endif` is accepted as a synonym for `end`. |
| Loop | `:( for item in collection ): … :( end ):` | Parentheses around `item in collection` are accepted. `endfor` is accepted as a synonym for `end`. |
| Include | `:( include "id" ):` | Quotes are optional. Resolves a template by frontmatter `id` through the theme's parent chain. |

The three terminators `end`, `endif` and `endfor` are lexed by one pattern, `(endif|endfor|end)`, so they are interchangeable. Using the matching one is still recommended for readability.

### Literals

| Kind | Pattern |
|---|---|
| Boolean true | `true`, `True`, `TRUE`, `yes`, `Yes`, `YES` |
| Boolean false | `false`, `False`, `FALSE`, `no`, `No`, `NO` |
| Integer | `[0-9]+` |
| Float | `[0-9]+\.[0-9]+` |
| Hexadecimal | `0x[0-9a-fA-F]+` |
| Octal | `0[0-9]+` |
| String | double-quoted `"…"` |
| Identifier | `[a-zA-Z_][a-zA-Z0-9_.]*` — dots are part of the identifier, so `site.pages.children` is one token |
| Array element | `identifier[expression]` — the index is **zero-based**. A negative index raises an error; an index past the end returns the whole array instead of one element. |

### Operators

| Group | Operators |
|---|---|
| Comparison | `==` `!=` `<` `<=` `>` `>=` |
| Logical | `&&` `\|\|` |
| Arithmetic (additive) | `+` `-` |
| Arithmetic (multiplicative) | `*` `/` `%` |
| Bitwise | `<<` `>>` `&` `\|` `^` |
| Assignment | `=` |
| Grouping | `( … )` |

The lexer also accepts the unary prefixes `!` and `~`, but in beta 75.1 a unary expression always evaluates to `0`/`false` regardless of its operand (the `UnaryOperation` class never evaluates its inner expression). Write `x == false` instead of `!x` until this is fixed. The operator strings `:=`, `!:=`, `~=` and `!~=` appear in the engine's operator table but no lexer token produces them, so they cannot be used.

### Evaluation

**Binary.** How the engine evaluates what it parsed:

- **No operator precedence.** An expression is parsed as *primary operator rest*, where *rest* is parsed recursively as a whole expression. Operators therefore group from the right: `a - b - c` is `a - (b - c)` and `a * b + c` is `a * (b + c)`. Use parentheses for anything with more than one operator.
- **The left operand decides the type.** For `==`, `!=`, `<`, `<=`, `>` and `>=` the engine looks at the type of the left operand only. If it is an integer, both sides are read as integers; a float, both as floats; a boolean, both as booleans; a string, both as strings (compared with `isEqualToString:`). Any other type (colour, image, link, array, dictionary, nil) falls back to object equality, and the ordering operators return `false` for it. So `"10" == 10` is true when the literal is on the right (the string `"10"` reads as the integer 10) but compares strings when the number is on the left.
- **Arithmetic result type** also follows the left operand: a colour gives colour math, an integer gives integer math, a float gives float math. A string or anything else on the left of `+`, `-`, `*` or `/` renders as an empty string, so `+` is **not** string concatenation; use `@concat`. Integer division and `%` by zero give `0`; float division by zero gives `inf`/`nan`.
- **Logical operators do not short-circuit.** `&&` and `||` evaluate both sides and combine them bitwise, which is equivalent to logical and/or for boolean results.
- **Truthiness** of a property in `if`: a missing key or nil is false; numbers and booleans use their value; a string is true when its text parses as true (`YES`, `true`, `1`, …, the Cocoa `boolValue` rules); a string literal in the template is true only if it parses as a non-zero integer; arrays, dictionaries, colours, images and links are always true. An arithmetic expression is true when its integer value is non-zero.
- **Missing keys** resolve in this order: the template context, the export context, a property of the current node, then the node's property defaults. If none matches the value is nil, which renders as an empty string and is false in conditions.
- **Type coercion when reading numbers.** An identifier read as an integer or float uses the value's `integerValue`/`floatValue` when it has one; a value with neither (an array, a dictionary, a colour) reads as `1`. A string literal reads as an integer with `atoi` rules, so `"3px"` is `3` and `"abc"` is `0`.
- **Errors** in expressions raise exceptions with named domains (`OperationException`, `ValueTypeException`, `ArrayIdentifierExpressionException`, `NumberValueException`, `FunctionException`); the message text is the one shown in the lists above.

### Functions

Functions are called as `@name(arguments)`. The lexer also accepts `#name(arguments)` for the same functions. Arguments are full expressions separated by commas. Fourteen functions are registered; their observed behaviour in beta 75.1:

| Function | Behaviour (decompiled) |
|---|---|
| `@concat(a, b, …)` | Renders every argument and joins the results with no separator. This is also what every function does with its arguments unless noted below. |
| `@count(a, b, …)` | Adds up, for each argument, the array or dictionary count or the string length. An empty string counts 0; a value with neither count nor length counts 1. Returns an integer. |
| `@length(a, …)` | Registered, but it has no implementation of its own and behaves exactly like `@concat`. Use `@count` for lengths. |
| `@lowerCase(…)` / `@upperCase(…)` / `@titleCase(…)` | Concatenate the arguments, then lower-case, upper-case or capitalise each word (`localizedCapitalizedString`). |
| `@deleteLastPathComponent(…)` | Concatenate, then drop the last path component. |
| `@appendPathComponent(a, b, …)` | Joins the string value of each argument with `/`, as `stringByAppendingPathComponent:` does. |
| `@markdown(…)` | Concatenate, then render the text as Markdown. Empty input gives an empty string. |
| `@round(n)` | Exactly one argument. Rounds to the nearest integer, halves away from zero. Returns an integer. |
| `@ceiling(n)` / `@floor(n)` | Exactly one argument. Round up / round down to an integer. |
| `@precision(n, digits)` | Exactly two arguments. Formats the float `n` with `digits` decimal places; `digits` must be 0–64. |
| `@number(value, options)` | Exactly two arguments; otherwise behaves like `@concat`. Renders `value` with `options` as its render properties. `options` is normally an identifier that resolves to a dictionary (for example a frontmatter block); a non-dictionary is wrapped as `{"string": …}`. |
| `@color(value, options)` | Same shape as `@number`. The colour renderer reads `allowsOpacity`, `prefix`, `operator` and `operand` from `options`, which is why the Sample theme declares a `colorProps` frontmatter block. |
| `@prefix(a, …, p)` | Two or more arguments. Concatenates all arguments except the last and prepends the rendered last argument, or `#` if it renders empty. With fewer than two arguments it behaves like `@concat`. Intended for building `#rrggbb` colour strings. |
| `@comment(…)` | Always renders an empty string. Use it to leave notes in a template. |

### Values

Rendered values have a type: nil, boolean, integer, float, string, array, dictionary, colour, image and link. Booleans render through `trueValue`/`falseValue` strings as in the Stack API. Colours render as CSS `rgb()`/`rgba()` strings with an optional prefix and support `+`, `-`, `*` and `/` colour math against numbers or other colours. Links expose `href` and the full anchor tag. Arrays, dictionaries and nil have fallback string renderings when output directly.

### Loop variables

Inside a loop, `__index` holds the iteration index, starting at **0**. Each iteration renders the body in a child context that adds two keys, the loop variable and `__index`, so both are visible to included templates. A collection that is nil produces no iterations; a single non-collection value is treated as a one-item collection. A nil item inside a collection is an error ("loop value undefined").

### Frontmatter parsing

**Binary:** Frontmatter is read by a YAML parser (libyaml). The template body starts after the closing `---` line, with leading whitespace removed. A template without frontmatter is used whole. The template's `id` is the frontmatter `id`; when that is missing the file name without extension is used instead.

Two frontmatter keys change how the body is parsed: `tagBegin` and `tagEnd` replace the `:(` and `):` delimiters for that template only. Characters in them are escaped, so any punctuation is safe. Templates included from this one keep their own delimiters.
