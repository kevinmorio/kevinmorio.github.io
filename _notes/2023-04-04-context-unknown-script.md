---
layout: post
title:  "ConTeXt: unknown script 'mtx-context.lua' or 'mtx-mtx-context.lua'"
permalink: date
date:   2023-04-04 22:00:12 +0100
---

After updating the TexLive installation on macOS, I got the following error when trying to build a ConTeXt project.

```
resolvers       | caches | path '/Users/kevin/.texlive2023/texmf-var' created
mtxrun          | unknown script 'mtx-context.lua' or 'mtx-mtx-context.lua'
```

To fix the error, run `mtxrun --generate` which will generate the file database.

## References

- [TeX - LaTeX - Stack Exchange](https://tex.stackexchange.com/questions/474246/what-is-the-mtxrun-error-about-context-lua-and-how-to-fix-it)
- [mtxrun(1) man page](https://manpages.debian.org/testing/context/mtxrun.1.en.html)
