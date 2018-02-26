---
layout: post
title: Markdown rendering issues on Hackage
image: /img/hackage_logo.png
show-avatar: true
---

Just a quick post on some Markdown rendering issues I recently ran into on Hackage. They were very annoying as the markup was properly rendered on github, and I could not spot any obvious problems.

## 1. First header is rendered as the original markup

You get

```
### Synopsis
```

instead of

### Synopsis

**Solution**: there is an invisible Unicode byte order mark in the beginning of the file. Seemingly Hackage does not like Unicode.

## 2. Subsequent lists are merged

You have a markup like this:

```
* list1_item1
* list1_item2

Something in between

* list2_item1
* list2_item2 

Something to the end
```

But you get something like this:

- list1_item1
- list1_item2  Something in between
- list2_item1
- list2_item2  Something to the end

**Solution**: you have Windows line endings. Obviously Hackage does not like Windows line endings.

