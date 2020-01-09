---
title: How to target modern browsers only with Next.js
date: "2019-05-22T20:32:55.000Z"
description: "Guide on correctly setting up your browserlist in your Next.js projects"
---

At work, we still support IE 11 (and it seems like Microsoft plans on supporting it until 2025, so yay!). Rebel that I am, in my side projects I target only modern browsers (by modern, I mean any browser that can run ES2015+ code).

Thanks to babel, today this is as easy as specifying the list of browsers you want to target. In your Next.js project, create a file called .babelrc (if you don't already have one), and add this to it:

```json
{
  "presets": [
    [
      "next/babel",
      {
        "preset-env": {
          "useBuiltIns": false,
          "targets": "Chrome >= 60, Safari >= 10.1, iOS >= 10.3, Firefox >= 54, Edge >= 15"
        }
      }
    ]
  ]
}
```

This piece of code extends the default Next.js babel preset with the browser targets you specified. The browsers listed in the targets property above all support most of the modern code you will be writing, so the final size of the code you ship should be quite a bit smaller.

It's time we stop shipping more code to all browsers only to support a small percentage of those that are stuck in the past.

In case you want to go all the way and support both old and new browsers (by having two separate builds), I would recommend further reading here: [https://philipwalton.com/articles/deploying-es2015-code-in-production-today/](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)
