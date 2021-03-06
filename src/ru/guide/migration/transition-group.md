---
title: Transition Group Root Element
badges:
  - breaking
---

# {{ $frontmatter.title }} <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

`<transition-group>` no longer renders a root element by default, but can still create one with the `tag` prop.

## Синтаксис в 2.x

In Vue 2, `<transition-group>`, like other custom components, needed a root element, which by default was a `<span>` but was customizable via the `tag` prop.

```html
<transition-group tag="ul">
  <li v-for="item in items" :key="item">
    {{ item }}
  </li>
</transition-group>
```

## Синтаксис в 3.x

In Vue 3, we have [fragment support](fragments.md), so components no longer _need_ a root node. Consequently, `<transition-group>` no longer renders one by default.

- If you already have the `tag` prop defined in your Vue 2 code, like in the example above, everything will work as before
- If you didn't have one defined _and_ your styling or other behaviors relied on the presence of the `<span>` root element to work properly, simply add `tag="span"` to the `<transition-group>`:

```html
<transition-group tag="span">
  <!-- -->
</transition-group>
```

## См. также

- [Some transition classes got a rename](transition.md)
