---
name: Capability Flags
syntax: "%supportsPreprocessor%"
group: special
description: Whether the running Stacks supports the preprocessor, compiler and Pro API.
available: 15
---

### Usage

```html
%supportsPreprocessor%
%supportsCompiler%
%supportsProApi%
```

### Description

Stacks 6 exposes three capability flags describing the host application: whether it supports the template `preprocessor`, the `compiler`, and the Pro API (`proApi`). They mirror the plist keys of the same names that a stack can declare.

> **Unconfirmed:** the rendered value (`true`/`false`, `1`/`0`) has not been verified. Test before branching on these in a conditional.

> Introduced in Stacks 6.0 (Stack API version 1500).
