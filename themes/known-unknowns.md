---
title: Theme API — Known Unknowns
layout: page
api_section: theme
back:
  - top: /themes
links:
  - title: Runtime Properties
    url: /themes/runtime
  - title: Inheritance
    url: /themes/inheritance
  - title: Reference Themes
    url: /themes/reference-themes
---

The available schema is explicitly inferred and open-ended. The following areas need additional runtime tests or confirmation from the Stacks developers.

Items marked **resolved** were answered by inspecting the Stacks 6.0 beta 75.1 application (see the **Binary** label on the [Theme API]({% include docs-root.html %}/themes/) page); the answers now live on the linked pages.

## Inheritance

- The independent effect of theme-level `inheritsFrom`.
- Whether controls, frontmatter or other properties merge across the `parent` chain.
- Behaviour when a parent is missing or circular inheritance is declared. (The engine has explicit renderer-loop detection, so a cycle is detected rather than looping forever; the user-visible result is unknown.)
- Whether parent resolution has version constraints. **Partly resolved:** `minThemeAPIVersion`/`maxThemeAPIVersion` gate the theme itself; see [Info.plist]({% include docs-root.html %}/themes/plist).

## Template language

- **Resolved:** the function list, operator set, literal forms, statement forms and terminator synonyms; see [Templates]({% include docs-root.html %}/themes/templates#expression-grammar).
- **Resolved (decompiled, beta 75.1):** truthiness, missing-property lookup, operand coercion, operator grouping (no precedence), the unary operators (`!`, `~`, currently inert), the function argument shapes, `__index` starting at 0, zero-based array subscripts and the `tagBegin`/`tagEnd` frontmatter override; see [Evaluation]({% include docs-root.html %}/themes/templates#evaluation). These describe the beta and should be re-checked on the release build.
- Escaping filters for HTML, attributes, URLs, CSS and JavaScript. (Boolean, string-encoding and link values exist as value types; no escaping function is registered.)
- How exceptions raised by expressions (`OperationException` and friends) surface to the user in Edit, Preview and Publish.
- Error reporting for malformed YAML.
- Whether duplicate template IDs in one bundle have deterministic precedence. (Templates are collected into a dictionary keyed by `id` while walking the `Templates` folder recursively; the last file wins, but directory enumeration order is not specified.)

## Runtime objects

- Complete schemas for `site`, `page`, navigation pages, breadcrumbs and theme objects. **Partly resolved:** see the additional keys on [Runtime Properties]({% include docs-root.html %}/themes/runtime#additional-runtime-keys).
- Which properties are stable public API and which are debugging implementation details.
- **Resolved:** the Theme API version is `themeAPIVersion` and Stacks 6.0 reports `60000`.
- Availability differences between Edit, Preview and Publish.

## Assets and libraries

- Complete library descriptor fields accepted specifically by themes. (The stack loader reads `name`, `closure`, `minVersion`, `maxVersion`; themes are loaded by the same code.)
- Which libraries are included in Edit mode.
- Deduplication and version-conflict behaviour.
- Asset collision behaviour across themes, layouts and stacks.

## Layouts

- Exact precedence when both a layout and theme provide page-level structure. (Layouts and themes share one renderer protocol; a layout has a fallback renderer and its own `layoutCssPath`/`layoutJsPath`/`layoutPhpPath`.)
- The complete list of layout content insertion points. **Resolved for themes:** the seven `content.*` slots are the complete set.
- Whether layout templates participate in the same named-template inheritance chain.

## Development lifecycle

- Which `.devstackstheme` changes hot-reload reliably.
- Which metadata changes require reselecting the theme or reopening the project.
- Compatibility guarantees across Theme API runtime versions.

## How to document new findings

For each test, record:

1. Stacks version and build.
2. macOS version.
3. Minimal theme bundles used by the test.
4. Edit, Preview and Publish results.
5. Generated HTML/CSS/JavaScript where relevant.

Once behaviour is repeatable, move it from this page into the appropriate reference page and label it **Observed** until it becomes formally specified.
