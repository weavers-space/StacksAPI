---
name: Maximum Instances per Page
key: pageMaxInstances
type: Number
group: sub-stacks
description: Limit how many copies of the stack a page may contain.
links:
  - top: /plist
---

Limits the number of instances of this stack that can be placed on one page.

> **Unconfirmed:** the key is read by the Stacks 5 and Stacks 6 loaders (it is one of the slice properties alongside `childOf` and `inline`), but the user-visible behaviour when the limit is reached has not been verified.
