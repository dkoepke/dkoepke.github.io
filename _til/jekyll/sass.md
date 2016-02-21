---
title: "Automatic SASS Compilation"
tags: [jekyll, github pages, sass]
category: jekyll
---

Create a directory named `_sass` with your SASS includes in it. In a directory that is exported (e.g., `assets/css`), create an `.scss` file (**without a leading underscore**). Jekyll will automatically compile it into a CSS file. An import like `@import "normalize";` will automatically import `_sass/_normalize.scss`.
