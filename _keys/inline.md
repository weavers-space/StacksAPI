---
name: Inline
key: inline
type: Boolean
group: sub-stacks
description: Disable wrapper divs around a stack
available: 7

---

Disable the standard wrapper `<div>` elements that Stacks places around a stack instance.

When `inline` is `true`, Stacks publishes only the content produced by the stack's templates and does not wrap it in the usual outer/inner container elements. Use this for stacks that must emit a bare fragment, such as a layout placeholder or an element that has to be a direct child of its parent in the DOM.

> Note: Without the wrapper elements the `%id%`, `%outerId%`, `%innerId%` and `%mezId%` templates have nothing to attach to. Do not rely on them in an inline stack.

The built-in Stacks 6 **Content** layout placeholder (`com.yourhead.stack.layout.content`) ships with `inline` set to `true`.
