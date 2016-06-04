---
layout: "post"
title: "Customizations for linter-markdown"
date: "2016-06-04 14:39"
---

When linter-markdown is installed, it will create a .remarkrc file at `~/.atom/packages/linter-markdown/.remarkrc` but at least in my installation, this file wouldn't be used.  I had to copy it to `~/.remarkrc` to get my customizations to take affect.

Once the file is in place, I applied the following two customizations:

~~~~ bash
"maximum-line-length": false,
"first-heading-level": false
~~~~

This prevents the linter from complaining about lines being over 80 characters and also from complaining when you use an H2 header without an H1 header before it.  I do this because with the theme that I use, the post title is already an H1 so I don't want the actual headers within the post body to be the same as the title.
