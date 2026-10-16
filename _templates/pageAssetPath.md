---
name: Page Asset Path
syntax: "%pageAssetPath%"
group: paths
description: The path to assets published with the current page.
available: 15
---

### Usage

```html
%pageAssetPath%
```

### Description

Returns the path to the folder that holds the assets published alongside the current page. In Stacks 6 this is the explicit name for the page-scoped half of [`%assetPath%`]({% include docs-root.html %}/templates/assetPath); use [`%siteAssetPath%`]({% include docs-root.html %}/templates/siteAssetPath) for assets declared with `siteAsset`.

> Introduced in Stacks 6.0 (Stack API version 1500).
