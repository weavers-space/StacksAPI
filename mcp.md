---
title: Stacks MCP Server
layout: page
api_section: stack
back:
  - top: /
links:
  - top: /stacks
  - top: /themes
---

Stacks 6 ships a local [Model Context Protocol](https://modelcontextprotocol.io) server, `stacks-mcp`, inside the application bundle at `Stacks.app/Contents/Resources/stacks-mcp`. It lets an AI agent or script inspect and edit the project that is open in Stacks. The server speaks MCP over stdio and talks only to the local Stacks application; it does not reach the network.

**Observed:** Stacks 6.0 beta 75.1 identifies the server as `stacks-native-agent` version `0.2.0`. It implements `tools/list` and `tools/call` only; `resources/list` and `prompts/list` return "Unknown method". Listing tools works without Stacks running, but calling them requires Stacks to be running with a project open.

## Registering the server

Stacks bundles a Codex plugin manifest (`Resources/AI/Stacks Codex Marketplace`) whose `.mcp.json` is:

```json
{
  "mcpServers": {
    "stacks": {
      "command": "/Applications/Stacks.app/Contents/Resources/stacks-mcp",
      "startup_timeout_sec": 30
    }
  }
}
```

The same command works for any MCP client that launches stdio servers, including Claude Code (`claude mcp add stacks -- /Applications/Stacks.app/Contents/Resources/stacks-mcp`).

## Object model

Source-list items are Pages, Partials, Layouts and Images, identified by `uid`. A placed stack inside a page editor is a different object (a node) with its own `uid`. Use the `stacks_get_item` / `stacks_set_item_properties` pair for source-list items and the `stacks_get_page_stack_snapshot`, `stacks_get_stack_properties` and `stacks_set_stack_properties` tools for placed stacks. Most tools take an optional `projectUid`; when it is omitted the front project is used.

## Observed with a project open

**Observed** against Stacks 6.0 beta 75.1 with a `.stacks` project open, using read-only calls.

**Envelope.** Every tool returns one text content block containing JSON. Success is `{"ok": true, "result": {…}}`; failure is `{"ok": false, "error": {"message": "…"}}` with `isError` set on the MCP result. Observed messages include `item not found`, `stack not found` and `missing stack match`.

**Projects.** `stacks_list_projects` returns `projects[]` with `uid`, `filePath` and `displayName`. `stacks_project_snapshot` returns `project`, `root` and `selection`. The root item has `kind: stacks.project.item.root` and five groups: `PAGES`, `LAYOUTS`, `IMAGES`, `ASSETS` and `PARTIALS`. Item kinds seen: `stacks.project.item.page`, `.layout`, `.stackstheme`, `.image`, `.partial`, `.group`. Each item carries `uid`, `title`, `subtitle`, `childCount` and `children`. Installed themes appear in `LAYOUTS` with their bundle identifier as `uid`.

**Installed stacks.** `stacks_list_stacks` returns `stacks[]` with `stackUid`, `title`, `subtitle`, `group` and `builtIn`. The 25 built-in entries include the layout, theme and content stacks (`com.yourhead.stacks.theme.base`, `com.yourhead.stack.layout.content`, `text`, `html`, `image`, …).

**Placed stacks.** `stacks_get_page_stack_snapshot` returns `page` and a `stacks[]` tree. Each node has `uid`, `libraryUid` (the stack identifier, empty for untitled inner stacks), `libraryTitle`, `displayName`, `userTitle`, `nid`, `type` (`stack`), `path` (for example `Home /  Container[1] / 1 Column[1]`), `ordinalAmongSameType`, `isSelected`, `recentlyAdded` and `children`. With `includeProperties: true` each node also has `properties[]`; `includeDebug` added nothing on the pages tested. The `page` argument is an object, `{"uid": …}` or `{"title": …}`.

**Property records** (also returned by `stacks_get_stack_properties`) have `uid`, `title`, `group`, `type`, `scope`, `valueKeyPath`, `value` and `metadata`. Stacks' own default controls use uids and groups prefixed `default###` (for example `default###backgroundColor`, group `default###Background`); a stack's custom controls use the plist key as `uid` and `custom.<key>` as `valueKeyPath`. `type` is the control family plus a component count: `""` for Stacks defaults, `input`, `multi-4` (text area), `number`, `number-2` … `number-4`, `slider`, `checkbox`, `button-2`/`button-3` (segmented), `select`, `color`, `color-2` … `color-4`, `link`, `image`, `detail`. A `select` record adds `options[]` (`index`, `title`, `value`) and `selection`. `metadata` holds `enabled`, `isClear`, `tip`, `componentCount`, `subtitles`, `radio`, `minimum`, `maximum`, `increment`, `round`, `units`, `ticks`, `snap` and `allowsOpacity`.

**Values.** Colours are `{"colorSpace": "srgb", "red", "green", "blue", "alpha"}`. Links are `{"kind": "remote", "href", "attributes"}` or `{"kind": "local", "uid", "attributes"}` where `uid` is a page. Images are `{"imageUid", "fileName", "title"}`. Multi-component controls return arrays.

**Matching.** `stacks_resolve_stack_reference` and `stacks_select_stack` take `match` as an object with any of `libraryTitle`, `userTitle`, `libraryUid`, `ordinal`, `nearText`, `ancestor`, `selected` and `recentlyAdded`; a string is rejected with `missing stack match`. A match that finds nothing returns `stack: null` and an empty `candidates` list rather than an error. Matching on `libraryTitle` is exact, and several bundled titles begin with a space (`" Container"`, `" PaySnap"`), so prefer `libraryUid`.

**Slots.** `stacks_get_stack_image_slots` returns `contentImages[]` (one-based `imageIndex`) and `imageProperties[]` (`propertyUid`, `title`, `isClear`). `stacks_get_stack_text_areas` returns `textAreas[]` with one-based `textIndex`.

## Tools

Parameters marked with `*` are required.

### `stacks_ping`

Check that the Stacks native agent bridge is reachable.

_No parameters._

### `stacks_list_projects`

List open Stacks projects.

_No parameters._

### `stacks_list_stacks`

List installed stacks that can be added to a page.

_No parameters._

### `stacks_project_snapshot`

Return the item tree and current selection for an open Stacks project.

| Parameter | Type | Description |
|---|---|---|
| `projectUid` | string |  |

### `stacks_get_selection`

Return the current page-list selection.

| Parameter | Type | Description |
|---|---|---|
| `projectUid` | string |  |

### `stacks_get_item`

Return one source-list Stacks item (Page, Partial, Layout, or Image) by uid. Do not use this for a placed page stack/YHNode; use stacks_get_page_stack_snapshot or stacks_get_stack_properties.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |

### `stacks_select_page`

Select and reveal one page in the page list by uid.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |

### `stacks_get_page_stack_snapshot`

Return a page's placed-stack tree in layout order. A page can be identified by uid or title.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `includeProperties` | boolean |  |
| `includeDebug` | boolean |  |
| `projectUid` | string |  |
| `page` | object |  |

### `stacks_resolve_stack_reference`

Resolve a placed stack reference on one page. Returns one stack or candidate stacks for an ambiguous match.

| Parameter | Type | Description |
|---|---|---|
| `match`* | object | Any of `libraryTitle`, `userTitle`, `libraryUid`, `ordinal`, `nearText`, `ancestor`, `selected`, `recentlyAdded`. |
| `pageUid` | string |  |
| `projectUid` | string |  |
| `page` | object | `{"uid"}` or `{"title"}`. |

### `stacks_select_stack`

Select one placed stack by uid or deterministic stack match in a page editor.

| Parameter | Type | Description |
|---|---|---|
| `match` | object |  |
| `pageUid` | string |  |
| `uid` | string |  |
| `projectUid` | string |  |
| `page` | object |  |

### `stacks_set_item_properties`

Set allowed properties on a source-list Stacks item (Page, Partial, Layout, or Image) by uid. Do not use this for a placed page stack/YHNode; use stacks_set_stack_properties instead.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |
| `properties`* | object |  |

### `stacks_add_page`

Add a page to a Stacks project. If parentUid is omitted, the page is added at the top level.

| Parameter | Type | Description |
|---|---|---|
| `title` | string |  |
| `projectUid` | string |  |
| `parentUid` | string |  |

### `stacks_delete_page`

Delete one page by uid.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |

### `stacks_move_page`

Move one page inside, before, or after another page. With no target page, inside moves it to the top-level page list.

| Parameter | Type | Description |
|---|---|---|
| `placement` | string: `inside`, `before`, `after` |  |
| `uid`* | string |  |
| `projectUid` | string |  |
| `overPageUid` | string |  |

### `stacks_add_partial`

Add an empty editable Partial to a Stacks project. For the normal workflow that converts placed stacks into a Partial, use stacks_create_partial_from_stacks instead.

| Parameter | Type | Description |
|---|---|---|
| `title` | string |  |
| `projectUid` | string |  |

### `stacks_create_partial_from_stacks`

Create a Partial from placed stacks on one page, replacing those stacks with the new Partial reference. This follows Stacks' normal Create Partial workflow. stackUids may be in any order; Stacks uses page order.

| Parameter | Type | Description |
|---|---|---|
| `pageUid`* | string |  |
| `title` | string |  |
| `stackUids`* | array |  |
| `projectUid` | string |  |

### `stacks_add_layout`

Add a default editable Layout with its normal content stack.

| Parameter | Type | Description |
|---|---|---|
| `title` | string |  |
| `projectUid` | string |  |

### `stacks_delete_source_item`

Delete one Partial, Layout, or Image from the source list by uid. Pages use stacks_delete_page.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |

### `stacks_move_source_item`

Reorder one Partial, Layout, or Image in its source-list tab. beforeUid places it before a sibling; omit beforeUid to move it to the end. Pages use stacks_move_page.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |
| `beforeUid` | string |  |

### `stacks_duplicate_item`

Duplicate an existing Stacks item by uid.

| Parameter | Type | Description |
|---|---|---|
| `uid`* | string |  |
| `projectUid` | string |  |

### `stacks_import_file`

Import a local file URL into a Stacks project. Images become project images and other supported files become assets.

| Parameter | Type | Description |
|---|---|---|
| `fileUrl`* | string |  |
| `projectUid` | string |  |
| `select` | boolean |  |

### `stacks_add_stack`

Add an installed stack to a Stacks page. If pageUid is omitted, the selected page is used.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `stackUid`* | string |  |
| `overStackUid` | string |  |
| `projectUid` | string |  |
| `placement` | string: `inside`, `before`, `after` |  |

### `stacks_add_stacks`

Add a nested batch of installed stacks to a Stacks page, with per-stack property settings.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `overStackUid` | string |  |
| `placement` | string: `inside`, `before`, `after` |  |
| `projectUid` | string |  |
| `stacks`* | array |  |

### `stacks_delete_stack`

Delete one placed stack by uid.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `projectUid` | string |  |
| `uid`* | string |  |

### `stacks_move_stack`

Move one placed stack inside, before, or after another stack. pageUid identifies the destination page; with no target stack, inside moves it to that page's root.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `uid`* | string |  |
| `overStackUid` | string |  |
| `projectUid` | string |  |
| `placement` | string: `inside`, `before`, `after` |  |

### `stacks_get_stack_properties`

Return the configurable properties for a placed stack by uid. Links are returned as {kind: remote, href, attributes} or {kind: local, uid, attributes}; colors as {colorSpace: srgb, red, green, blue, alpha}. A color-2, color-3, color-4, number-2, number-3, or number-4 property returns an array with that many values.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `projectUid` | string |  |
| `uid`* | string |  |

### `stacks_set_stack_properties`

Set configurable properties on a placed stack by uid. A link property accepts a URL string, {kind: remote, href, attributes}, or {kind: local, uid, attributes}; local uid must identify a page, image, or asset in this project. A color property accepts {colorSpace: srgb, red, green, blue, alpha} with 0...1 components, or #RRGGBB / #RRGGBBAA. A color-2, color-3, color-4, number-2, number-3, or number-4 property accepts an array containing exactly two, three, or four matching values.

| Parameter | Type | Description |
|---|---|---|
| `properties`* | object |  |
| `pageUid` | string |  |
| `uid`* | string |  |
| `projectUid` | string |  |

### `stacks_set_stack_boolean`

Set one one-based Boolean control in a placed stack button or checkbox property. When the property is a radio group, setting a control to true clears the other controls.

| Parameter | Type | Description |
|---|---|---|
| `value`* | boolean |  |
| `pageUid` | string |  |
| `uid`* | string |  |
| `propertyUid`* | string |  |
| `projectUid` | string |  |
| `index`* | integer |  |

### `stacks_set_stack_date`

Set a placed stack date property from a timezone-qualified ISO-8601 date-time string, for example 2026-08-30T17:00:00Z.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `uid`* | string |  |
| `propertyUid`* | string |  |
| `projectUid` | string |  |
| `date`* | string |  |

### `stacks_clear_stack_property`

Clear one placed stack property using the Slice-defined clear value, the same behavior as the property's Clear control.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `uid`* | string |  |
| `propertyUid`* | string |  |
| `projectUid` | string |  |

### `stacks_get_stack_image_slots`

List a placed stack's direct image-content slots and image properties. Use the returned one-based imageIndex or propertyUid for image changes.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `projectUid` | string |  |
| `uid`* | string |  |

### `stacks_set_stack_image_property`

Set or replace a placed stack image property from an existing project imageUid or a local image fileUrl. A local fileUrl creates a new project image; exactly one source is required.

| Parameter | Type | Description |
|---|---|---|
| `fileUrl` | string |  |
| `pageUid` | string |  |
| `uid`* | string |  |
| `propertyUid`* | string |  |
| `projectUid` | string |  |
| `imageUid` | string |  |

### `stacks_clear_stack_image_property`

Clear a placed stack image property without deleting the underlying project image.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `uid`* | string |  |
| `propertyUid`* | string |  |
| `projectUid` | string |  |

### `stacks_set_stack_content_image`

Set or replace a placed stack's direct image-content child from an existing project imageUid or a local image fileUrl. A local fileUrl creates a new project image; exactly one source is required.

| Parameter | Type | Description |
|---|---|---|
| `fileUrl` | string |  |
| `pageUid` | string |  |
| `uid`* | string |  |
| `imageIndex`* | integer |  |
| `projectUid` | string |  |
| `imageUid` | string |  |

### `stacks_clear_stack_content_image`

Remove a placed stack's direct image-content child without deleting the underlying project image.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `uid`* | string |  |
| `imageIndex`* | integer |  |
| `projectUid` | string |  |

### `stacks_set_stack_select`

Choose a placed stack popup-menu option by its user-visible title, one-based index, first, or last. Inspect the select property's options before choosing when titles are unclear.

| Parameter | Type | Description |
|---|---|---|
| `selection`* | object |  |
| `pageUid` | string |  |
| `uid`* | string |  |
| `propertyUid`* | string |  |
| `projectUid` | string |  |

### `stacks_get_stack_text_areas`

List a placed stack's direct text areas in order. textIndex is one-based.

| Parameter | Type | Description |
|---|---|---|
| `pageUid` | string |  |
| `projectUid` | string |  |
| `uid`* | string |  |

### `stacks_set_stack_text_area`

Replace one placed stack text area with plain text. textIndex is one-based.

| Parameter | Type | Description |
|---|---|---|
| `textIndex`* | integer |  |
| `pageUid` | string |  |
| `uid`* | string |  |
| `projectUid` | string |  |
| `text`* | string |  |
