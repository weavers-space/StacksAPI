---
name: Minimum Theme API Version
key: minThemeAPIVersion
type: Number
group: version
description: The minimum Theme API version (themes and layouts).
links:
  - top: /plist
  - top: /themes/plist
---

The minimum Theme API version required by a theme or layout bundle. When the running Stacks reports an older Theme API, the add-on is refused with the message "This add-on works only with Theme API version N or greater".

Stacks reads `minThemeAPIVersion`, `minThemeApiVersion` and `minthemeapiversion` as the same key. Stacks 6.0 reports Theme API version 60000. See the [Theme API plist reference]({% include docs-root.html %}/themes/plist).

> Introduced in Stacks 6.0.
