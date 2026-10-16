---
title: Theme Bundles
layout: page
api_section: theme
back:
  - top: /themes
links:
  - title: Getting Started
    url: /themes/getting-started
  - title: Info.plist Reference
    url: /themes/plist
  - title: Templates
    url: /themes/templates
---

## Required structure

**Schema:** A theme is a macOS-style bundle with the extension `.stackstheme` and these required paths:

```text
Example.stackstheme/
└── Contents/
    ├── Info.plist
    └── Resources/
        └── Templates/
```

The schema identifies the following optional resource locations:

```text
Contents/Resources/assets
Contents/Resources/Templates/theme_overrides
Contents/Resources/Templates/basic_theme_overrides
```

**Binary:** Templates are loaded recursively from `Contents/Resources/Templates`, and a template's identity is its frontmatter `id`. The `theme_overrides` and `basic_theme_overrides` folders are therefore only a naming convention used by the bundled Base and Sample themes to group the templates a child theme is expected to override (`theme_css`, `theme_js`, `theme_meta`, `theme_foot`). Any folder name works.

## Info.plist

`Contents/Info.plist` identifies the bundle as a theme and declares metadata, controls, assets and libraries. See [Theme Info.plist]({% include docs-root.html %}/themes/plist).

## Templates

Every file in `Contents/Resources/Templates` may define a named template through YAML frontmatter. Template files can use any extension. The bundled themes conventionally use `.html`, `.css` and `.js` extensions because those extensions make the source easier to understand and edit.

Template identity comes from its frontmatter `id`, not its filename.

## Icons

Bundled themes use:

```xml
<key>icon</key>
<string>icon</string>
```

and include large standard and Retina PNGs in `Contents/Resources`. Existing Stacks icon naming conventions should be supported when producing distributable themes.

## Development bundles

**Observed:** Rename a working theme to `.devstackstheme` while developing it. Stacks reloads many resource changes without requiring the theme to be reinstalled.

Changes to bundle metadata or an already-instantiated theme may still require one of the following:

- Switching to another theme and back.
- Closing and reopening the project.
- Restarting Stacks when a completely new bundle is first introduced.

Do not distribute a development bundle as the finished product.

## Assets and libraries

Assets are files or folders copied into published output. Libraries are named shared resources. Their plist declarations follow the same general API used by ordinary stacks; see [Theme Info.plist]({% include docs-root.html %}/themes/plist#assets-and-libraries).
