---
layout: post
title: "Arranging PDF pages in a grid"
date: 2023-04-30 14:32:38 +0200
categories: posts
---

[Zoom Notes][ZN], a note-talking app for iOS, has a feature that allows you to arrange pages of a PDF in a grid.
Instead of scrolling and jumping through the document, you can freely explore it on a canvas.
The spatial layout allows you to annotate and connect ideas from multiple pages in an easy way.
Zoom out and you get a bird’s-eye view on your marginalia and highlights.

{:style="text-align:center; margin: 3rem 0;"}
![Zoom Notes PDF Import](/assets/images/zoom-notes-pdf-import.png){:style="max-width: 24rem;"}

Unfortunately, this feature is not available on all the different platforms and devices I use on a regular basis.
For a platform-agnostic solution, we can rearrange the pages of a PDF directly and then use any PDF reader of our choice.

A handy tool that let's you accomplish this (and a lot more interesting things) is [pdfjam][pj].
It is a script around the [pdfpages][pp] LaTeX package.
Want to split, join, select or layout pages of PDF documents? pdfjam can do it.

### pdfjam

We can arrange the pages of a standard PDF document in a grid with `n` columns and `m` rows per page as follows.

``` shell
$ pdfjam in.pdf --nup nxm --outfile out.pdf
```

{:style="text-align:center; margin: 3rem 0;"}
![Zoom Notes PDF Import](/assets/images/pdfjam-pdf-grid.png){:style="max-width: 24rem;"}

pdfjam uses A4 as its default paper size.
If we want a different size, we can use the `--paper` or `--papersize` options.

If the margins are too small or too large, we can shrink or extend them using

``` shell
--trim '<left> <bottom> <right> <top>' --clip
```

where `<left>` can be `2cm` to shrink the left margin by two centimeters or `-5cm` to extend it by five centimeters.

**Remark:** It's advisable to keep the number of columns and rows of the grid reasonable. Turning a 900-page book into a single 30×30 grid page easily overloads most PDF readers or at least degrades the performance.

Explanations of the other interesting features and options for pdfjam can be found in the help (`pdfjam --help`) and the documentation of the [pdfpages][pp] package.

## References

- [pdfjam on GitHub][pj]
- [pdfpages LaTeX package][pp]

[ZN]: http://www.zoom-notes.com
[pj]: https://github.com/rrthomas/pdfjam
[pp]: https://www.ctan.org/tex-archive/macros/latex/contrib/pdfpages