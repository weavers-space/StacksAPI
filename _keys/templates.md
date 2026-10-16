---
name: Template Files
key: templates
type: Array
value: An array of template dictionaries.
description: Declare your template files here.
group: content
available: 5

links:
  - top: /plist
  - plist: keys/assets
  - plist: keys/libraries

---

An array of template dictionaries.

#### Assets vs. Templates vs. Libraries

Assets, Templates, and Libraries are similar. Each can contain content and files to be published. But each imply different behavior and should be used differently.

##### Assets
Assets are NOT processed for templates. They are NOT included in edit-mode. Assets should be used whenever possible. They are simple and low-cost.
##### Libraries
Libraries are NOT processed for templates. Some libraries are included in edit-mode. Some libraries are built-in to stacks (e.g. jQuery and FontAwesome). Libraries are shared between all stacks on the site/page.  Use a library whenever you need to share common content between stacks.
##### Templates
Templates ARE processed for template variables. Some templates are included in edit-mode. Since templates must be processed you should include as few templates as possible -- move as much of your content to assets or libraries whenever you can.


#### Template Dictionary

Each line in the templates array should be a dictionary. And each dictionary should represent a single template file that should be processed.

> Each template file and each template variable within each file adds a significant performance cost to your stack. To ensure your stack has the best performance strive to include as few template files as possible with as few templates variables as possible.

The template dictionary can include the following keys:

##### Filename -- key: `filename`
The filename of the template. Do not include the path to the file.

##### File type -- key: `type`
This can be any of the following:
	
- `html` -- This content is included in the index.html page content. You can choose where on the page the content will be included using the scope (see below). `html` content is included in Edit, Preview, and Publish modes.
- `css` -- The style content for your stack. It will be grouped with other style content and published in a separate file.
- `js` -- Javascript content is NOT included in edit mode. By default JS content is wrapped in a closure and grouped together into a single JS file.
- `php` -- PHP is only included when publishing. It is NOT combined into a single file, but published in individual files.
- `file` -- The file-type template is a catch-all. You should use it only when one of the other template types doesn't fit. (Example: JSON Config files, Raw Text files, etc.) File type content can be any sort of UTF-8 text based file that can be template processed (if you'd like to include non-template processed files, or non-text files, they should be [assets]({% include docs-root.html %}/keys/assets) instead). Keep in mind that each file will be processed for templates. Include only what you need. Use assets for all that you can.

##### Scope -- key: `scope`
The scope of a template determins where on the page/file it will be published, as well as which template variables will be processed.

- `each` -- *default* -- The each scope publishes content for each stack instance on the page. All template variables will be processed.
- `page` -- The page scope publishes content ONLY ONCE for all stack instances on the page. Some template variables will be processed. Page-Scoped template variables will be processed, as well as any variable that is static for the entire page (e.g. version numbers and file-paths). Each template variable lists the scopes that it will be published in.
- `first` -- (Stacks 6, Stack API 1500) -- Like `page`, the content is published ONLY ONCE per page, but it is rendered with the context of the **first instance** of the stack on the page. Instance variables such as [`%id%`]({% include docs-root.html %}/templates/id), `%innerId%` and `%parentId%` therefore resolve to that first instance. Use it for one-time setup that still needs to know which stack it belongs to.
- `site` -- Like the page scope, the site scope will only be published once for the entire site, into the Stacks common folder (`site-assets`). Site-scoped templates cannot use page or instance variables. **Binary:** `css`, `js` and `php` templates accept this scope; `file` templates are only exported for `each`, `first` and `page`.

###### HTML positions -- These scopes are limited to `html` templates

Each position exists in a `page` flavour (rendered once with the page context) and, in Stacks 6, a `first` flavour (rendered once with the first instance's context). The `page*` scopes except `pageSuffix` also exist in Stacks 5.

| Page scope | First scope (Stacks 6) | Position |
|---|---|---|
| `pagePrefix` | `firstPrefix` | Immediately before the stack content, inside the Stacks container. In the published page this is the first place a PHP template can declare variables for the stack instances that follow. |
| `pageHeadAbove` | `firstHeadAbove` | In the `<head>` tag above all other content, immediately AFTER the call to jQuery, jQueryUI, and FontAwesome. |
| `pageHeadBelow` | `firstHeadBelow` | In the `<head>` tag below all js declarations. |
| `pageBodyAbove` | `firstBodyAbove` | Immediately BEFORE all other stack content. This is the absolute highest place in the `<body>` tag that Stacks has access to. |
| `pageBodyBelow` | `firstBodyBelow` | Immediately AFTER all other stack content. This is the absolute lowest place in the `<body>` tag that Stacks has access to. Some Javascript works best that way. |
| `pageSuffix` | `firstSuffix` | Immediately after the stack content, inside the Stacks container. `pageSuffix` is new in Stacks 6. |

**Binary:** the complete scope list accepted by Stacks 6.0 beta is `each`, `first`, `page`, `site`, `pagePrefix`, `pageSuffix`, `pageHeadAbove`, `pageHeadBelow`, `pageBodyAbove`, `pageBodyBelow`, `firstPrefix`, `firstSuffix`, `firstHeadAbove`, `firstHeadBelow`, `firstBodyAbove` and `firstBodyBelow`. An unknown scope string is treated as "unknown" and the template is not published. Stacks 5.3 accepts `each`, `page`, `site`, `pagePrefix`, `pageHeadAbove`, `pageHeadBelow`, `pageBodyAbove` and `pageBodyBelow`.

> Observed: in a Partial, `pagePrefix` PHP runs in a different file from the partial's `each` content, so variables set in a prefix are not visible to instances inside the Partial. Use `firstPrefix` or put the shared code in a `php` template instead.

#### Published locations

**Observed** with the Weaver's Space *Stacks API Test* stack on Stacks 6.0 beta. Paths are relative to the published site root; `files/` is the page's files folder.

| Content | Published to |
|---|---|
| `each`, `first` and `page` scoped `css` / `js` | `files/stacks.css`, `files/stacks.js` (one combined file per page) |
| `each`, `first` and `page` scoped `php` | `files/stacks.php` |
| `site` scoped `css` / `js` / `php` | `site-assets/stacks.css`, `site-assets/stacks.js`, `site-assets/stacks.php` |
| `page` or `first` scoped `file` | `files/<filename>` |
| `each` scoped `file` | `files/<innerId>_<filename>` -- one copy per instance, prefixed with the instance's inner element id |
| Page asset | `files/<filename>` |
| Site asset (`siteAsset`) | `site-assets/stacks/<bundle identifier>/<filename>` |
| Custom library | `site-assets/libraries/<name>/<filename>` |
| Built-in libraries | `site-assets/libraries/fontawesome/font-awesome.min.css`, `site-assets/libraries/jquery3/jquery3.min.js`, … |

When the stack is placed in a **Layout** rather than a page, the combined CSS and JavaScript move to `site-assets/layouts/<layout id>/stacks.css` and `.js`, and the page title, navigation title, file name and path variables are not available. On the current beta, layout-placed `php` templates and page assets are written to `site-assets/stacks/<bundle identifier>/` and page-scoped and each-scoped `file` templates are not exported at all; treat those as open issues.
