---
title: The Property List
layout: page
api_section: stack
back:
  - top: /bundle
links:
  - plist: keys/assets
  - plist: keys/libraries
  - plist: keys/templates
---

The Info.plist file within a stack defines the basic attributes of the stack like its title and version number, the custom properties the stack will display when selected, and file names of each of the templates and assets.



{% assign groups = site.keys | sort: 'group' %}
{% assign currentGroup = 'basic' %}

<ul>
<h3>{{ currentGroup | capitalize }}</h3>
{% for property in groups %}

  {% if property.group %}
    {% if currentGroup != property.group %}
      {% assign currentGroup = property.group %}
<br>      
<br>      
<h3>{{ currentGroup | capitalize }}</h3>

    {% endif %}
  {% endif %}

{% if property.available == site.version %}
    <li class='new'>
{% else %}
    {% if property.deprecated == site.version %}
        <li class='new deprecation'>
    {% else %}
        <li>
    {% endif %}
{% endif %}

<a href='{% include docs-root.html %}{{ property.url }}'>{{ property.name }}</a> &mdash; <code>{{ property.key }}</code> &mdash; {{ property.description }}</li>

{% endfor %}
</ul>

### Keys read by Stacks that are not yet documented

The Stacks 5.3 and 6.0 loaders also read the following keys. Their behaviour has not been confirmed, so they are listed here rather than given reference pages. Authoring tools should preserve them.

- `proApi` and `compiler` — declared capabilities that pair with the [`%supportsProApi%` and `%supportsCompiler%`]({% include docs-root.html %}/templates/supports) templates.
- `libraryImageFile` — an alternative library icon file.
- `minRWVersion` — a RapidWeaver version floor used by some Stacks 5 built-in stacks.
- `maxChildCount` — read next to `addTypes`/`allowedTypes` in Stacks 6 (compare `maxChildren` on [`%slice%`]({% include docs-root.html %}/templates/slice)).
