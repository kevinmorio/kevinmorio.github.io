---
layout: post
title:  "Combine images on the command line"
date:   2023-04-07 12:48:02 +0100
categories: posts
---

[ImageMagick's][IM] and [GraphicMagic's][GM] `convert` command can be used to combine multiple images into a single larger one.
With the `-append` flag, the images are combined from top-to-bottom, with `+append` from left-to-right.
The images can be of different formats.
The output format is inferred from the extension of the out-file.

``` shell
$ convert A.png B.png C.jpg -append ABC.png
```

{:style="text-align:center;"}
![ABC](../assets/images/ABC.png)

``` shell
$ convert A.png B.png C.jpg +append A-B-C.png
```

{:style="text-align:center;"}
![A-B-C](../assets/images/A-B-C.png)

The `-append` and `+append` flags can also be nested to lay out the images in a grid.

``` shell
$ convert A.png B.png -append \( C.jpg D.jpg -append \) +append AC-BD.png
```

{:style="text-align:center;"}
![AC-BD](../assets/images/AC-BD.png)

If the dimensions of the images are not the same, they are expanded to fit using the background color.

``` shell
$ convert -background "#333333" ABC.png D.jpg +append ABC-D.png
```

{:style="text-align:center;"}
![ABC-D](../assets/images/ABC-D.png)

## References

- [convert(1) man page](https://manpages.debian.org/testing/graphicsmagick-imagemagick-compat/convert.1.en.html#append)

[IM]: https://imagemagick.org/index.php
[GM]: http://www.graphicsmagick.org
