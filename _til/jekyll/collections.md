---
title: "Collections in Jekyll / GitHub Pages"
tags: [jekyll, github pages, collections]
category: jekyll
date: 2016-02-18
---

Collections are a new, experimental feature in Jekyll. I'm using them to create a Today I Learned section of my personal GitHub Pages site. To do it, I've put this in my `_config.yml`:

```yaml
collections:
  til:
    output: true            # Output the documents in the generated site
    permalink: /til/:path/  # Make the URLs look like this
    layout: til-post        # The layout to use for the TIL entries
```

I then created a directory called `_til` with the content of documents I want to put in this collection. Each file is a Markdown file with YAML front matter:

```yaml
---
title: "What I Learned"
tags: [foo, bar, baz]
category: whatever
---

This is the content that will be rendered.
```

I can place these in subdirectories if I want--it doesn't matter.
