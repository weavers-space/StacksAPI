---
name: Page CSS Path
syntax: "%pageCssPath%"
group: paths
description: The path to the page-scoped generated CSS file.
available: 15
---

### Usage

```html
%pageCssPath%
```

### Description

The path of the CSS file Stacks generates for the current page. In Stacks 6 the generated files exist at page, layout and site scope; `%pageCssPath%` names the page one explicitly, while [`%cssPath%`]({% include docs-root.html %}/templates/cssPath) keeps its Stacks 5 meaning. The companions are `%pageJsPath%` and `%pagePhpPath%`.

> Introduced in Stacks 6.0 (Stack API version 1500).
