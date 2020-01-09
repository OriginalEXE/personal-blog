---
title: Server rendering emotion with Nuxt
date: "2018-11-15T21:47:27.000Z"
description: "How to use emotion and Nuxt together"
---

I have had a few flings with CSS-in- JS so far but never anything serious, but this week I decided to commit to using emotion in my side project. I am using Nuxt and have written all of my styles with emotion (well, all except the global ones, which I decided to load from an external stylesheet).

It all went reasonably well, until I tried to generate my website for production (using `nuxt generate`). And there it was, the infamous FOUC, or flash of unstyled content. Turns out that emotion will not automatically output the css needed for the components out of which the initial view is composed, it only outputs them on the client side, resulting in a brief flash.

After hours of searching, I managed to solve it:

First of all, npm install the `emotion-server` package: `npm install --save emotion-server`

Then, require it in your `nuxt.config.js` file: `const { renderStylesToString } = require ('emotion-server');`

This will allows us to use a very cool function `renderStylesToString` which takes HTML and inlines the CSS necessary for each component before it is needed. We now just need to hook into a nuxt hook that will allows us to modify the HTMl response from the server.

Here is what those might look like (added to the object you are exporting from your `nuxt.config.js` file):

```js
    hooks: {
      render: {
        route (url, result) {
          const withCss = renderStylesToString (result.html);

          result.html = withCss;
        },
      },
      generate: {
        page (page) {
          const withCss = renderStylesToString (page.html);

          page.html = withCss;
        },
      },
    },
```

I am using two different hooks because I am statically exporting my website, and a render hook is not called with `nuxt generate`.

That's it! I have lost too much time finding this, so now it's here, ready to save the next person some time :)

**Bonus hint:** Due to the way `renderStylesToString` inserts the CSS, you should avoid using `:first-child` , `:nth-child()` etc. in your css as all those style tags in your HTML might cause a flash once they are hydrated on the client and moved to the `<head>`.
