---
name: Maximum Theme API Version
key: maxThemeAPIVersion
type: Number
group: version
description: The maximum Theme API version (themes and layouts).
links:
  - top: /plist
  - top: /themes/plist
---

The maximum Theme API version a theme or layout bundle supports. When the running Stacks reports a newer Theme API, the add-on is refused with the message "This add-on works only with Theme API version N or less".

Stacks reads `maxThemeAPIVersion`, `maxThemeApiVersion` and `maxthemeapiversion` as the same key.

> Introduced in Stacks 6.0.
