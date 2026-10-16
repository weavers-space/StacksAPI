---
name: Site CSS Path
syntax: "%siteCssPath%"
group: paths
description: The path to the site-scoped generated CSS file.
available: 15
---

### Usage

```html
%siteCssPath%
```

### Description

The path of the CSS file that Stacks 6 generates once for the whole site (the output of `site`-scoped CSS templates and themes). The companions are `%siteJsPath%` and `%sitePhpPath%`.

> Introduced in Stacks 6.0 (Stack API version 1500).
