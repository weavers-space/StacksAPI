---
name: Maximum Stacks API Version
key: maxStackAPIVersion
type: Number
group: version
description: The maximum API version.
links:
  - top: /plist
  - plist: keys/minAPIVersion
---

The maximum Stack API version your stack supports. When the running Stacks reports a newer API, the stack is refused with the message "This add-on works only with Stack API version N or less".

Stacks reads `maxStackAPIVersion`, `maxStackApiVersion` and `maxstackapiversion` as the same key. The matching minimum is [`minAPIVersion`]({% include docs-root.html %}/keys/minAPIVersion), which is also accepted as `minStackAPIVersion`.

> Warning: Only set a maximum when a stack is known to break on a newer API. Stacks 6.0 reports Stack API version 1500, so a Stacks 5 stack with `maxStackAPIVersion` of 13 will not load in Stacks 6.
