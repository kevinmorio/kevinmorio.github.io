---
layout: post
title:  "Different ways of exporting Kindle annotations"
date:   2023-01-31 23:10:56 +0100
categories: ebooks
---

When exporting my annotations from my Kindle the other day, I noticed some interesting differences about what information is included in the process depending on the method used.

## Share to email on Kindle Scribe

When opening the annotations on the Kindle Scribe, you can either send them to your account's email address or up to five other email addresses you can enter. I received my annotations almost instantly.

We receive a nicely formatted PDF containing the highlights and corresponding notes including handwritten ones. Independent of the display choice of the position on the Kindle, the annotations use page references. If you have used different highlight colors when annotating on the mobile app, they are not reported here. Also no added date is exported in this case.

When the export limit (in this case 10%) is reached (after ~6969 words), only notes and bookmarks are exported.

**Summary**

- **Includes handwritten notes:**: yes
- **Includes highlight color**: no
- **Includes added date**: no
- **Position type:** page
- **Exported words:** ~6969

## Transfer "My Clippings.txt"

When you connect your Kindle to your computer, you find the `My Clippings.txt` file in the `documents` directory.

Since the `My Clippings.txt` is a regular text file, no handwritten notes are included. Even no indication that a handwritten note exists at all. The color of the highlights is again not reported. However, we get the time when the annotation was added to the second and also both a page and location position.

When the export limit is reached (here after just ~5553 words), only notes and bookmarks are exported. This is indicated by the line

``` text
 <You have reached the clipping limit for this item>
```

For some reason, the file contains several duplicates of almost identical highlights which only differ by a few words. This may explain why about 1400 fewer words than in the previous case were exported.

**Summary**

- **Includes handwritten notes:**: no
- **Includes highlight color**: no
- **Includes added date**: yes
- **Position type:** page + location
- **Exported words:** ~5553

## Share on Kindle app (Android)

When opening the annotations on the Kindle app on Android, you can choose a citation style and export them by sharing a file.

We get a nicely formatted HTML file containing the highlights, notes, and bookmarks, but no handwritten notes. This is the only (official) export option that includes both the color of the highlights as well as the part and chapter title.

When the export limit is reached (here after ~6934 words), only notes and bookmarks are exported. The number of words exported is almost the same as in the first case. The minor difference is due to the extraction of the text from the PDF and HTML files.

**Summary**

- **Includes handwritten notes:**: no
- **Includes highlight color**: yes
- **Includes added date**: no
- **Position type:** part + chapter + page
- **Exported words:** ~6934
