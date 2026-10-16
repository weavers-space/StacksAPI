---
name: Asset Path by Name
syntax: "%asset=filename%"
group: paths
description: The published path of one declared asset file.
available: 15
---

### Usage

```html
%asset=<filename>%
```

 - `<filename>` — the `filename` of an entry in the plist [`assets`]({% include docs-root.html %}/keys/assets) array. Letters, digits, `.`, `_` and `-` are allowed.

### Description

Returns the published path of a single declared asset, resolving whether that asset is a page asset or a site asset (`siteAsset`) so the template does not need to choose between [`%assetPath%`]({% include docs-root.html %}/templates/assetPath) and [`%siteAssetPath%`]({% include docs-root.html %}/templates/siteAssetPath) itself. The output is the matching asset folder path, a `/`, and the file name.

**Binary:** the name is matched against the last path component of each declared asset, site assets first, then page assets. If it matches neither, the macro outputs the bare file name so the page still renders; check the published HTML for a relative name with no folder when an asset seems to be missing.

> Introduced in Stacks 6.0 (Stack API version 1500). Stacks 5 does not recognise this template and will publish it literally.

#### Example 1: link a stylesheet that may be a page or site asset

```html
<link rel="stylesheet" href="%asset=theme.css%">
```
