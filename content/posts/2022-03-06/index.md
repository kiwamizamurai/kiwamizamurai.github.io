---
title: "How to enable latex on PaperMod"
date: 2022-03-06T00:00:00+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Latex", "Katex"]
author: "kiwamizamurai"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Enable Latex by Katex"
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/kiwamizamurai/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---


# Abstract

I'll show you how to allow markdown to display latex. This website uses hugo and it does not have the ability to render latex by default.

`$e^2$`

In addition, I write down a way to solve a little latex problem.

# Content

## Setup

First of all, create a file `layouts/partials/extend_head.html` with the following content.

```html
{{ if or .Params.math .Site.Params.math }}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.15.2/dist/katex.min.css" integrity="sha384-MlJdn/WNKDGXveldHDdyRP1R4CTHr3FeuDNfhsLPYrq2t0UBkUdK2jyTnXPEK1NQ" crossorigin="anonymous">
<!-- The loading of KaTeX is deferred to speed up page rendering -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.15.2/dist/katex.min.js" integrity="sha384-VQ8d8WVFw0yHhCk5E8I86oOhv48xLpnDZx5T9GogA/Y84DcCKWXDmSDfn13bzFZY" crossorigin="anonymous"></script>
<!-- To automatically render math in text elements, include the auto-render extension: -->
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.15.2/dist/contrib/auto-render.min.js" integrity="sha384-+XBljXPPiv+OzfbB3cVmLHf4hdUFHlWNZN5spNQ7rmHTXpd7WvJum6fIACpNNfIR" crossorigin="anonymous"
    onload="renderMathInElement(document.body);"></script>

<!-- for inline -->>
<script>
document.addEventListener("DOMContentLoaded", function() {
    renderMathInElement(document.body, {
        delimiters: [
            {left: "$$", right: "$$", display: true},
            {left: "$", right: "$", display: false}
        ]
    });
});
</script>
{{ end }}
```

Then, add the parameter in order to enable MathJax on `config.yml` as follows:

```yml
params:
  math: true
```

That's all. Now I can use latex on hugo.

$$  \gamma^2+\theta^2=\omega^2 $$

## Problem

However, there is a tiny problem, which is that the writing style is a little bit different from pure latex on markdown.

> Many markdown processors use the underscore (_) to indicate italics (one at the beginning and one at the end of the text to be italicized). So when your math contains two underscores, Markdown removes them and inserts <em>...</em> tags (or something equivalent to that) before sending the page to the browser. MathJax doesn't process math that contains HTML tags, so the resulting (modified) math is not typeset.
> The usual solution is to use a backslash to prevent the underscore from being processed by Markdown, so use \_ in place of _ in your math. You may also need to double some backslashes (e.g., \\ may need to be entered as \\\\ in a Markdown document).

The quote comes from stackoverflow, namely I have to replace `_` with `\_`. There is one more thing, new line requires `\\\` instead of `\\`


# References

- [Using KaTeX in hugo](https://rkilingr.me/posts/katex-hugo/)
- [markdown latex driving me crazy with hugo on github](https://stackoverflow.com/questions/53587872/markdown-latex-driving-me-crazy-with-hugo-on-github)
- [How to enable Math Typesetting in PaperMod? · Issue #236 · adityatelange/hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod/issues/236)
- [FAQs · adityatelange/hugo-PaperMod Wiki](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#custom-head--footer)