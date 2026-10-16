---
name: Bundle Type
key: type
type: String
description: Whether the bundle is a stack, a theme or a layout.
links:
  - top: /plist
  - top: /themes/plist
---

Declares what kind of add-on the bundle is. Stacks 6 recognises:

- `stack` — the default when the key is absent. An ordinary stack.
- `theme` — a theme bundle (`.stackstheme`). See the [Theme API]({% include docs-root.html %}/themes/).
- `layout` — a layout placeholder stack. The built-in **Content** stack (`com.yourhead.stack.layout.content`) declares `type` of `layout` together with [`inline`]({% include docs-root.html %}/keys/inline).

```xml
<key>type</key>
<string>theme</string>
```

> Introduced in Stacks 6.0. Stacks 5 ignores the key.
